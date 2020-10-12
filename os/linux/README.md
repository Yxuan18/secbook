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

涉及配置文件为：`/etc/hosts.deny`与`/etc/hosts.allow`。



下面解释sshd\_config各项：

```text
## 重要 ##

#Port 22
设置sshd监听端口号，默认情况下为22，可以设置多个监听端口号，即重复使用Prot这个设置项。修改后记得重启sshd，以及在防火墙中添加端口。出于安全考虑，端口指定为小于等于65535，并且非22或22变种的值

#ListenAddress 0.0.0.0
#ListenAddress ::
设置sshd监听（绑定）的IP地址，0.0.0.0表示监听所有IPv4的地址。出于安全考虑，设置为指定IP地址，而非所有地址。::是IPv6的地址不需要改

#LoginGraceTime 2m 
设置指定时间内没有成功登录，将会断开连接，若无单位则默认时间为秒

#PermitRootLogin yes
是否允许root登录，默认是允许的，但建议设置为no

#PasswordAuthentication yes
是否使用密码验证。当然也可以设置为no，不使用密码验证，转而使用密钥登录

#PermitEmptyPasswords no
是否允许空密码的用户登录，默认为no，不允许

#PrintMotd yes
是否打印登录提示信息，提示信息存储在/etc/moed文件中

#PrintLastLog yes
显示上次登录信息。默认为yes

#UseDNS yes
一般来说为了要判断客户端来源是否正常合法，因此会使用DNS去反查客户端的主机名。但通常在内网互连时，设置为no，使连接快写

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



