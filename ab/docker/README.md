# Docker

## 一、docker-compose安装

###  1、步骤

Linux 上我们可以从 Github 上下载它的二进制包来使用，最新发行的版本地址：[https://github.com/docker/compose/releases](https://github.com/docker/compose/releases)。

运行以下命令以下载 Docker Compose 的当前稳定版本：

```bash
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

要安装其他版本的 Compose，请替换 1.24.1。

将可执行权限应用于二进制文件：

```bash
$ sudo chmod +x /usr/local/bin/docker-compose
```

创建软链：

```bash
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

测试是否安装成功：

```bash
$ docker-compose --version
cker-compose version 1.24.1, build 4667896b
```

**注意**： 对于 alpine，需要以下依赖包： py-pip，python-dev，libffi-dev，openssl-dev，gcc，libc-dev，和 make。

![](../../.gitbook/assets/image%20%28559%29.png)

### 2、基本使用

```bash
docker-compose build    #编译环境
docker-compose up -d    #编译环境并在后台执行服务
```

```text
docker-compose down
```

上述命令会执行如下几个动作：

* 关闭正在运行的容器
* 删除所有相关容器
* 移除NAT（docker-compose在运行的时候会创建一个NAT网段）

## 二、docker安装

### 1、使用官方安装脚本自动安装

安装命令如下：

```bash
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

也可以使用国内 daocloud 一键安装命令：

```bash
curl -sSL https://get.daocloud.io/docker | sh
```

### 2、手动安装

#### 卸载旧版本

Docker 的旧版本被称为 docker，docker.io 或 docker-engine 。如果已安装，请卸载它们：

```bash
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

当前称为 Docker Engine-Community 软件包 docker-ce 。

安装 Docker Engine-Community，以下介绍两种方式。

#### 使用 Docker 仓库进行安装

在新主机上首次安装 Docker Engine-Community 之前，需要设置 Docker 仓库。之后，您可以从仓库安装和更新 Docker 。

#### 设置仓库

更新 apt 包索引。

```bash
$ sudo apt-get update
```

安装 apt 依赖包，用于通过HTTPS来获取仓库: 

```bash
$ sudo apt-get install \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg-agent \
     software-properties-common
```

添加 Docker 的官方 GPG 密钥：

```bash
$ curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```

9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88 通过搜索指纹的后8个字符，验证您现在是否拥有带有指纹的密钥。 

```bash
$ sudo apt-key fingerprint 0EBFCD88
    
 pub   rsa4096 2017-02-22 [SCEA]
       9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
 uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
 sub   rsa4096 2017-02-22 [S]
```

使用以下指令设置稳定版仓库 

```bash
$ sudo add-apt-repository \
    "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/ \
   $(lsb_release -cs) \
   stable"
```

#### 安装 Docker Engine-Community

更新 apt 包索引。

```bash
$ sudo apt-get update
```

安装最新版本的 Docker Engine-Community 和 containerd ，或者转到下一步安装特定版本：

```bash
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

要安装特定版本的 Docker Engine-Community，请在仓库中列出可用版本，然后选择一种安装。列出您的仓库中可用的版本： 

```bash
$ apt-cache madison docker-ce

   docker-ce | 5:18.09.1~3-0~ubuntu-xenial | https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu  xenial/stable amd64 Packages
   docker-ce | 5:18.09.0~3-0~ubuntu-xenial | https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu  xenial/stable amd64 Packages
   docker-ce | 18.06.1~ce~3-0~ubuntu       | https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu  xenial/stable amd64 Packages
   docker-ce | 18.06.0~ce~3-0~ubuntu       | https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu  xenial/stable amd64 Packages
   ...

```

使用第二列中的版本字符串安装特定版本，例如 5:18.09.1~3-0~ubuntu-xenial。

```bash
$ sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
```

测试 Docker 是否安装成功，输入以下指令，打印出以下信息则安装成功: 

```bash
$ sudo docker run hello-world

 Unable to find image 'hello-world:latest' locally
 latest: Pulling from library/hello-world
 1b930d010525: Pull complete                                                                                                                                  Digest: sha256:c3b4ada4687bbaa170745b3e4dd8ac3f194ca95b2d0518b417fb47e5879d9b5f
 Status: Downloaded newer image for hello-world:latest


 Hello from Docker!
 This message shows that your installation appears to be working correctly.


 To generate this message, Docker took the following steps:
  1. The Docker client contacted the Docker daemon.
  2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
     (amd64)
  3. The Docker daemon created a new container from that image which runs the
     executable that produces the output you are currently reading.
  4. The Docker daemon streamed that output to the Docker client, which sent it
     to your terminal.


 To try something more ambitious, you can run an Ubuntu container with:
  $ docker run -it ubuntu bash


 Share images, automate workflows, and more with a free Docker ID:
  https://hub.docker.com/


 For more examples and ideas, visit:
  https://docs.docker.com/get-started/
```

###  3、使用 Shell 脚本进行安装

Docker 在 [get.docker.com](https://get.docker.com/) 和 [test.docker.com](https://test.docker.com/) 上提供了方便脚本，用于将快速安装 Docker Engine-Community 的边缘版本和测试版本。脚本的源代码在 docker-install 仓库中。 不建议在生产环境中使用这些脚本，在使用它们之前，您应该了解潜在的风险：

* 脚本需要运行 root 或具有 sudo 特权。因此，在运行脚本之前，应仔细检查和审核脚本。
* 这些脚本尝试检测 Linux 发行版和版本，并为您配置软件包管理系统。此外，脚本不允许您自定义任何安装参数。从 Docker 的角度或您自己组织的准则和标准的角度来看，这可能导致不支持的配置。
* 这些脚本将安装软件包管理器的所有依赖项和建议，而无需进行确认。这可能会安装大量软件包，具体取决于主机的当前配置。
* 该脚本未提供用于指定要安装哪个版本的 Docker 的选项，而是安装了在 edge 通道中发布的最新版本。
* 如果已使用其他机制将 Docker 安装在主机上，请不要使用便捷脚本。

本示例使用 [get.docker.com](https://get.docker.com/) 上的脚本在 Linux 上安装最新版本的Docker Engine-Community。要安装最新的测试版本，请改用 test.docker.com。在下面的每个命令，取代每次出现 get 用 test。

```bash
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

如果要使用 Docker 作为非 root 用户，则应考虑使用类似以下方式将用户添加到 docker 组：

```bash
$ sudo usermod -aG docker your-user
```

## 三、docker简单使用

### 安装

```bash
sudo apt-get install docker.io
yum -y install docker-io
```

### 启动docker服务

```bash
service docker start
```

### 常用命令：

| 释义 | 命令 |
| :---: | :--- |
| 搜索镜像 | docker search keyword |
| 下载镜像 | docker pull image\_name:version |
| 查看本地所有镜像 | docker images |
| 查看所有运行中容器 | docker ps |
| 查看所有容器 | docker ps -a |
| 启动一个容器 | docker run --name=rand\_name  -it -d image\_name sh |
| 进入一个容器 | docker exec 容器id bash |
| 以root的方式进入容器 | docker  exec -u root  -it   容器id  bash |
| 暂停一个容器 | docker stop 容器id |
| 移除一个容器 | docker rm 容器id |
| 移除一个镜像 | docker rmi 镜像id |
| 批量停止容器 | docker stop $\(docker ps -q\) |
| 批量移除容器 | ``docker rm `docker ps -a -q```  |
| 批量移除镜像 | ``docker rmi `docker images -q```  |

### docker 启动一个容器

```bash
docker run --name=test -p 2080:80 -p 2022:22 -it -d -v /test:/soft  镜像id
 #   --name给容器命名
 #   -d:让容器在后台运行。
 #   -P:将容器内部使用的网络端口映射到我们使用的主机上。（主机的端口：容器端口）
 #   -v：目录挂载，数据持久化 （主机目录:容器内目录）
```

### docker 容器封装成镜像

```bash
docker commit :从容器创建一个新的镜像。
# 语法
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
#OPTIONS说明：
#    -a :提交的镜像作者；
#    -c :使用Dockerfile指令来创建镜像；
#    -m :提交时的说明文字；
#    -p :在commit时，将容器暂停。
##  实例 
将容器a404c6c174a2 保存为新的镜像,并添加提交人信息和说明信息。
runoob@runoob:~$ docker commit -a "runoob.com" -m "my apache" a404c6c174a2  mymysql:v1 
sha256:37af1236adef1544e8886be23010b66577647a40bc02c0885a6600b33ee28057
```

### docker打包

```bash
docker save 镜像名:版本号 -o 打包压缩存放位置
#实例：
docker save memcached:1.4 -o /home/wg4a/xmglpt/memcached1.4.tar
docker解压镜像：
docker docker load -i memcached1.4.tar

将容器打包：
docker export 容器id> name.tar
```

### docker更换下载源

```text
docker exec -it 容器id bash
sed -i s@/deb.debian.org/@/mirrors.aliyun.com/@g /etc/apt/sources.list
```

### docker封装cms的基本思路

1.下载一个LAMP环境（不一定是LAMP根据实际情况pull）

```text
docker pull image_name:versionx
```

2 启动环境（根据实际情况选择参数，镜像id通过docker image 查看）

```text
dcoekr run -d  --name=lamp -p 80:80 -v /mysql_data:/var/lib/mysql  镜像id
```

3.进入容器（docker ps 查看容器id）

```text
docker exec -it 容器id bash
```

4.切换到web路径，使用wegt命令将cms添加的docker容器中，也可在第二步中直接挂在进去

5.cms的正常安装

6.退出容器，并暂停容器

7.将容器封装为镜像，如果有本地挂载，同时保存一份本地挂载的文件，以便复现时导入

8. 将容器镜像打包

###  dockerfile封装CMS基本思路

 1、编写Dockerfile

```bash
FROM ubuntu:18.04    # 
LABEL maintainer="phithon <root@leavesongs.com>"
```

2、运行命令封装为镜像

```bash
docker build -t 镜像名 .
```

## 四、常见问题

###  1、docker pull的时候出错

```text
error pulling image configuration: Get https://production.cloudflare.docker.com/
registry-v2/docker/registry/v2/blobs/sha256/1c/1cda43d811c8fb178d9b0aacbdfaaa4e4
f36489e2112558c84a97549f868585c/data?verify=1608172124-zf3G3JFGipnlE3G24jL3h8nIKg
w%3D: dial tcp: lookup production.cloudflare.docker.com: Temporary failure in name
 resolution
```

步骤如下：

1、编辑 /etc/resolv.conf 文件，在文件中添加如下内容：

```text
nameserver 8.8.8.8
```

![](../../.gitbook/assets/image%20%281069%29.png)

2、保存并推出，之后重启docker服务

###  2、docker pull: "No address associated with hostname"

```text
# 报错内容---ubuntu18.04
Error response from daemon: Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io: No address associated with hostname
```

解决方案如下：（参考[链接](https://www.cnblogs.com/wannengachao/p/12119840.html)）

重启网络与docker

```bash
systemctl daemon-reload
systemctl restart docker
```

![&#x6548;&#x679C;&#x56FE;](../../.gitbook/assets/image%20%281072%29.png)

###  3、docker的daemon.json

```bash
vim /etc/docker/daemon.json
-------------------------------
{
    "registry-mirrors": ["https://registry.docker-cn.com"]
}
-------------------------------
# 之后需要重启网络与docker服务
systemctl daemon-reload
systemctl restart docker.service
```

