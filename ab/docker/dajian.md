# Docker 私有仓库搭建

## 一、简介 <a href="#yi-jian-jie" id="yi-jian-jie"></a>

在 Docker 中，当我们执行 docker pull xxx 的时候 ，它实际上是从 registry.hub.docker.com 这个地址去查找，这就是Docker公司为我们提供的公共仓库。在工作中，我们不可能把企业项目push到公有仓库进行管理。所以为了更好的管理镜像，Docker不仅提供了一个中央仓库，同时也允许我们搭建本地私有仓库。这一篇介绍registry、harbor两种私有仓库搭建。

## 二、registry 的搭建 <a href="#er-registry-de-da-jian" id="er-registry-de-da-jian"></a>

### 1. 搭建 <a href="#1-da-jian" id="1-da-jian"></a>

Docker 官方提供了一个搭建私有仓库的镜像 **registry** ，只需把镜像下载下来，运行容器并暴露5000端口，就可以使用了。

```
docker pull registry:2
```

```
docker run -d -v /opt/registry:/var/lib/registry -p 5000:5000 --name myregistry registry:2
```

Registry服务默认会将上传的镜像保存在容器的/var/lib/registry，我们将主机的/opt/registry目录挂载到该目录，即可实现将镜像保存到主机的/opt/registry目录了。

浏览器访问http://127.0.0.1:5000/v2，出现下面情况说明registry运行正常。

![](<../../.gitbook/assets/image (990).png>)

### 2. 验证 <a href="#2-yan-zheng" id="2-yan-zheng"></a>

现在通过push镜像到registry来验证一下。

查看本地镜像：

```
$ docker images
REPOSITORY                                             TAG                 IMAGE ID            CREATED             SIZE
nginx                                                  latest              568c4670fa80        5 weeks ago         109MB
ubuntu                                                 latest              93fd78260bd1        7 weeks ago         86.2MB
elasticsearch                                          6.5.1               32f93c89076d        7 weeks ago         773MB
```

要通过docker tag将该镜像标志为要推送到私有仓库：

```
docker tag nginx:latest localhost:5000/nginx:latest
```

通过 docker push 命令将 nginx 镜像 push到私有仓库中：

```
docker push localhost:5000/nginx:latest
```

访问 [http://127.0.0.1:5000/v2/\_catalog](http://127.0.0.1:5000/v2/\_catalog) 查看私有仓库目录，可以看到刚上传的镜像了：

![](<../../.gitbook/assets/image (988).png>)

下载私有仓库的镜像，使用如下命令：

```
docker pull localhost:5000/镜像名:版本号
例如
docker pull localhost:5000/nginx:latest
```

## 二、harbor 的搭建 <a href="#er-harbor-de-da-jian" id="er-harbor-de-da-jian"></a>

docker 官方提供的私有仓库 registry，用起来虽然简单 ，但在管理的功能上存在不足。 Harbor是一个用于存储和分发Docker镜像的企业级Registry服务器，harbor使用的是官方的docker registry(v2命名是distribution)服务去完成。harbor在docker distribution的基础上增加了一些安全、访问控制、管理的功能以满足企业对于镜像仓库的需求。

### 1.搭建 <a href="#1-da-jian" id="1-da-jian"></a>

#### 下载 <a href="#xia-zai" id="xia-zai"></a>

地址：[https://github.com/goharbor/harbor/releases](https://github.com/goharbor/harbor/releases) 本文是有 v1.2.2

#### 配置 <a href="#pei-zhi" id="pei-zhi"></a>

解压下载的安装包 harbor-offline-installer-v1.2.2.tgz

```
tar -xvf harbor-offline-installer-v1.2.2.tgz
```

修改 harbor.cfg

```
#hostname 改为本地ip，非 Mac OS系统 可以不指定端口
hostname = 192.168.31.143:9090
#设置secretkey_path 的路径为 当前目录的data下
secretkey_path = ./data
```

**需要注意的是，非 Mac 用户只需要 修改 harbor.cfg 中的 hostname ，就可以直接通过./install.sh 就可以构建镜像，并把服务启动起来。不需要 secretkey\_path 和 下面 docker-compose.yml 的修改**

修改 docker-compose.yml

因为harbor使用了很多目录挂载，Mac有很多目录是不允许挂载的，所以如果是Mac用户，需要修改docker-compose.yml 中的挂载目录，修改后的 docker-compose.yml 如下：

```yaml
version: '2'
services:
  log:
    image: vmware/harbor-log:v1.2.2
    container_name: harbor-log 
    restart: always
    volumes:
      - ./log/:/var/log/docker/:z
    ports:
      - 127.0.0.1:1514:514
    networks:
      - harbor
  registry:
    image: vmware/registry:2.6.2-photon
    container_name: registry
    restart: always
    volumes:
      - ./data/registry:/storage:z
      - ./common/config/registry/:/etc/registry/:z
    networks:
      - harbor
    environment:
      - GODEBUG=netdns=cgo
    command:
      ["serve", "/etc/registry/config.yml"]
    depends_on:
      - log
    logging:
      driver: "syslog"
      options:  
        syslog-address: "tcp://127.0.0.1:1514"
        tag: "registry"
  mysql:
    image: vmware/harbor-db:v1.2.2
    container_name: harbor-db
    restart: always
    volumes:
      - ./data/database:/var/lib/mysql:z
    networks:
      - harbor
    env_file:
      - ./common/config/db/env
    depends_on:
      - log
    logging:
      driver: "syslog"
      options:  
        syslog-address: "tcp://127.0.0.1:1514"
        tag: "mysql"
  adminserver:
    image: vmware/harbor-adminserver:v1.2.2
    container_name: harbor-adminserver
    env_file:
      - ./common/config/adminserver/env
    restart: always
    volumes:
      - ./data/config/:/etc/adminserver/config/:z
      - ./data/secretkey:/etc/adminserver/key:z
      - ./data/:/data/:z
    networks:
      - harbor
    depends_on:
      - log
    logging:
      driver: "syslog"
      options:  
        syslog-address: "tcp://127.0.0.1:1514"
        tag: "adminserver"
  ui:
    image: vmware/harbor-ui:v1.2.2
    container_name: harbor-ui
    env_file:
      - ./common/config/ui/env
    restart: always
    volumes:
      - ./common/config/ui/app.conf:/etc/ui/app.conf:z
      - ./common/config/ui/private_key.pem:/etc/ui/private_key.pem:z
      - ./data/secretkey:/etc/ui/key:z
      - ./data/ca_download/:/etc/ui/ca/:z
      - ./data/psc/:/etc/ui/token/:z
    networks:
      - harbor
    depends_on:
      - log
      - adminserver
      - registry
    logging:
      driver: "syslog"
      options:  
        syslog-address: "tcp://127.0.0.1:1514"
        tag: "ui"
  jobservice:
    image: vmware/harbor-jobservice:v1.2.2
    container_name: harbor-jobservice
    env_file:
      - ./common/config/jobservice/env
    restart: always
    volumes:
      - ./data/job_logs:/var/log/jobs:z
      - ./common/config/jobservice/app.conf:/etc/jobservice/app.conf:z
      - ./data/secretkey:/etc/jobservice/key:z
    networks:
      - harbor
    depends_on:
      - ui
      - adminserver
    logging:
      driver: "syslog"
      options:  
        syslog-address: "tcp://127.0.0.1:1514"
        tag: "jobservice"
  proxy:
    image: vmware/nginx-photon:1.11.13
    container_name: nginx
    restart: always
    volumes:
      - ./common/config/nginx:/etc/nginx:z
    networks:
      - harbor
    ports:
      - 9090:80
      - 443:443
      - 4443:4443
    depends_on:
      - mysql
      - registry
      - ui
      - log
    logging:
      driver: "syslog"
      options:  
        syslog-address: "tcp://127.0.0.1:1514"
        tag: "proxy"
networks:
  harbor:
    external: false
```

通过运行 install.sh 构建镜像，并把服务启动起来：

```
./install.sh
```

### 2. 使用 <a href="#2-shi-yong" id="2-shi-yong"></a>

访问 [http://127.0.0.1:9090/](http://127.0.0.1:9090/) 如下：

![](<../../.gitbook/assets/image (986).png>)

默认 admin 用户的密码为 Harbor12345 ，可以在 harbor.cfg 进行修改。登录后如下：

![](<../../.gitbook/assets/image (989).png>)

图中的项目是之前上传的 ，新部署的 Harbor 登录后项目下是空的。

可以创建项目，创建用户，给项目分配用户等等，操作都很简单 。

### 3. 上传镜像 <a href="#3-shang-chuan-jing-xiang" id="3-shang-chuan-jing-xiang"></a>

首先登录私有仓库，可以使用 admin 用户 ，也可以使用我们自己创建的具有上传权限的用户：

```
docker login -u admin -p Harbor12345 127.0.0.1:9090
```

要通过docker tag将该镜像标志为要推送到私有仓库，例如：

```
docker tag nginx:latest 127.0.0.1:9090/library/nginx:latest
```

上传镜像：

```
docker push 127.0.0.1:9090/library/nginx:latest
```

访问 http://127.0.0.1:9090/harbor/projects ，在 library 项目下可以看见刚上传的 nginx镜像了：

![](<../../.gitbook/assets/image (992).png>)

\
&#x20;![](https://img2018.cnblogs.com/blog/528977/201903/528977-20190311112108481-2022563516.jpg)
