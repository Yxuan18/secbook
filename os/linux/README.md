# Linux

## 一、常见问题

###  1、更换APT源

1、此处以[清华源](https://mirror.tuna.tsinghua.edu.cn/help/ubuntu/)为例，将 `/etc/apt/sources.list`中的所有文本全部删除，然后更换为如下内容，保存退出即可

```text
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
```

更换后，更新效果图如下：

![](../../.gitbook/assets/image%20%28531%29.png)

### 2、为ROOT设置密码

1. 打开命令行，并`sudo su`切换至管理员权限
2. 输入命令：`passwd root`，之后设置密码即可

### 3、安装并开启SSH服务

 1、尝试开启服务，发现本地没有SSH服务端

![](../../.gitbook/assets/image%20%28533%29.png)

 2、使用命令下载SSH服务端

```text
apt install openssh-server
```

3、安装成功后，查看SSH服务状态

![](../../.gitbook/assets/image%20%28537%29.png)

```text
## 相关命令如下：

service ssh status    #查看SSH状态
service ssh start     #启动服务
service ssh stop      #停止服务
service ssh restart   #重启服务
```

### 4、SSH相关

 配置文件位于：`/etc/ssh/sshd_config`   ：

① 设置允许SSH远程登录：

![](../../.gitbook/assets/image%20%28538%29.png)

新增内容为：

```text
PermitRootLogin yes
```

 新增后，保存并退出，并重启SSH服务即可

![&#x8FDE;&#x63A5;&#x6210;&#x529F;](../../.gitbook/assets/image%20%28534%29.png)

② 限制可远程访问IP



下面解释各项：

```text

```

### 5、修改docker镜像源

 新建或打开`/etc/docker/daemon.json` ，在其中输入如下内容，保存并退出，且重启docker服务即可

```text
{
  "registry-mirrors": ["https://y0qd3iq.mirror.aliyuncs.com"]
}
```

![&#x6D41;&#x7A0B;](../../.gitbook/assets/image%20%28535%29.png)



### 6、7、8、9、



