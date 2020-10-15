# Docker容器环境检测方法【代码】

参考资料：

Determining if a process runs inside lxc/Docker

[http://stackoverflow.com/questions/20010199/determining-if-a-process-runs-inside-lxc-docker](http://stackoverflow.com/questions/20010199/determining-if-a-process-runs-inside-lxc-docker)

## 第1章 背景 <a id="&#x7B2C;1&#x7AE0;-&#x80CC;&#x666F;"></a>

现在有研究表明，人们目前有使用Docker进行恶意代码重现工作的倾向。Docker的反检测可分为三部分：CPU反检测，内存反检测和存储数据反检测。该技术利用了AUFS的层叠镜像技术，使得我们的Docker反检测技术可以很容易地实现在镜像的一层中——反检测层。这样，只需要将反检测层动态地部署到Docker容器中，即可实现对Container环境的CPU、内存和存储的封装，从而遮蔽掉Container特征。这样的话，该容器中的恶意代码也就无法检测其自身是否在Container中运行了。

## 第2章 cgroup方法 <a id="&#x7B2C;2&#x7AE0;-cgroup&#x65B9;&#x6CD5;"></a>

在Host和Container中执行cat /proc/1/cgroup命令的结果是不同的，可以利用这一点进行判断。（或者cat /proc/self/cgroup命令也行）

在Host中执行cat /proc/1/cgroup命令的结果：

```text
root@ubuntu:~
10:hugetlb:/
9:cpuset:/
8:memory:/
7:cpu,cpuacct:/
6:perf_event:/
5:blkio:/
4:net_cls,net_prio:/
3:freezer:/
2:devices:/
1:name=systemd:/
```

在Container中执行cat /proc/1/cgroup命令的结果：

```text
root@33b328a4095b:/go/src/github.com/docker/docker
10:hugetlb:/docker/33b328a4095bc92e9d427bc9f702d87b7f20c3ea6c12137a35783bc4fd942478
9:cpuset:/docker/33b328a4095bc92e9d427bc9f702d87b7f20c3ea6c12137a35783bc4fd942478
8:memory:/docker/33b328a4095bc92e9d427bc9f702d87b7f20c3ea6c12137a35783bc4fd942478
7:cpu,cpuacct:/docker/33b328a4095bc92e9d427bc9f702d87b7f20c3ea6c12137a35783bc4fd942478
6:perf\_event:/docker/33b328a4095bc92e9d427bc9f702d87b7f20c3ea6c12137a35783bc4fd942478
5:blkio:/docker/33b328a4095bc92e9d427bc9f702d87b7f20c3ea6c12137a35783bc4fd942478
4:net\_cls,net\_prio:/docker/33b328a4095bc92e9d427bc9f702d87b7f20c3ea6c12137a35783bc4fd942478
3:freezer:/docker/33b328a4095bc92e9d427bc9f702d87b7f20c3ea6c12137a35783bc4fd942478
2:devices:/docker/33b328a4095bc92e9d427bc9f702d87b7f20c3ea6c12137a35783bc4fd942478
1:name=systemd:/docker/33b328a4095bc92e9d427bc9f702d87b7f20c3ea6c12137a35783bc4fd942478
```

## 第3章 find docker方法 <a id="&#x7B2C;3&#x7AE0;-find-docker&#x65B9;&#x6CD5;&#x539F;&#x521B;"></a>

在Host和Container中的文件系统中，有关于docker的文件、目录有很大差别。因此可以通过枚举文件系统中关于docker的文件，进行判断。

在Host中执行find / -name docker命令的结果：

```text
root@ubuntu:~
/usr/bin/docker
/var/lib/docker
/var/lib/docker/aufs/diff/4d465ea0a041fe7ea0dfff03f1bfed15524ea35354b6c23e5aaeb05bf5341039/go/src/github.com/docker
/var/lib/docker/aufs/diff/0933def894f49be2f894483f42f31ba26110a8a7a68fc294b8b47d4fee5ab46a/usr/bin/docker
/var/lib/docker/aufs/diff/0933def894f49be2f894483f42f31ba26110a8a7a68fc294b8b47d4fee5ab46a/run/docker
/var/lib/docker/aufs/diff/0933def894f49be2f894483f42f31ba26110a8a7a68fc294b8b47d4fee5ab46a/etc/docker
/run/docker
/etc/init.d/docker
/etc/bash_completion.d/docker
/etc/docker
/etc/default/docker
/etc/apparmor.d/cache/docker
/etc/apparmor.d/docker
/sys/fs/cgroup/hugetlb/docker
/sys/fs/cgroup/cpuset/docker
/sys/fs/cgroup/memory/docker
/sys/fs/cgroup/cpu,cpuacct/docker
/sys/fs/cgroup/perf_event/docker
/sys/fs/cgroup/blkio/docker
/sys/fs/cgroup/net_cls,net_prio/docker
/sys/fs/cgroup/freezer/docker
/sys/fs/cgroup/devices/docker
/sys/fs/cgroup/systemd/docker
```

在Container中执行find / -name docker命令的结果：

```text
root@33b328a4095b:/go/src/github.com/docker/docker# find / -name docker     
/etc/bash_completion.d/docker
/go/src/github.com/docker
/go/src/github.com/docker/docker
/go/src/github.com/docker/docker/docker
/go/src/github.com/docker/docker/builder/dockerfile/parser/testfiles/docker
/go/src/github.com/docker/docker/contrib/init/sysvinit-debian/docker
/go/src/github.com/docker/docker/contrib/init/sysvinit-redhat/docker
/go/src/github.com/docker/docker/contrib/completion/bash/docker
/go/src/github.com/docker/docker/vendor/src/github.com/docker
/var/lib/docker
/docker-py/docker
```

## 第4章 环境变量方法 <a id="&#x7B2C;4&#x7AE0;-&#x73AF;&#x5883;&#x53D8;&#x91CF;&#x65B9;&#x6CD5;&#x539F;&#x521B;"></a>

在Host和Container中执行env命令查看环境变量的结果是不同的，可以利用这一点进行判断。（或者cat /proc/1/environ命令也可）

在Host中执行env命令的结果：

```text
root@ubuntu:~# env
XDG_VTNR=7
XDG_SESSION_ID=c2
CLUTTER_IM_MODULE=xim
XDG_GREETER_DATA_DIR=/var/lib/lightdm-data/root
SESSION=ubuntu
TERM=xterm-256color
SHELL=/bin/bash
VTE_VERSION=4002
WINDOWID=58720266
UPSTART_SESSION=unix:abstract=/com/ubuntu/upstart-session/0/1451
GNOME_KEYRING_CONTROL=
GTK_MODULES=unity-gtk-module
USER=root
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.jpg=01;35:*.jpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.axv=01;35:*.anx=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.axa=00;36:*.oga=00;36:*.spx=00;36:*.xspf=00;36:
XDG_SESSION_PATH=/org/freedesktop/DisplayManager/Session0
XDG_SEAT_PATH=/org/freedesktop/DisplayManager/Seat0
SSH_AUTH_SOCK=/run/user/0/keyring/ssh
DEFAULTS_PATH=/usr/share/gconf/ubuntu.default.path
XDG_CONFIG_DIRS=/etc/xdg/xdg-ubuntu:/usr/share/upstart/xdg:/etc/xdg
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
DESKTOP_SESSION=ubuntu
QT_IM_MODULE=ibus
QT_QPA_PLATFORMTHEME=appmenu-qt5
PWD=/root
XDG_SESSION_TYPE=x11
JOB=unity-settings-daemon
XMODIFIERS=@im=ibus
GNOME_KEYRING_PID=
LANG=en_US.UTF-8
MANDATORY_PATH=/usr/share/gconf/ubuntu.mandatory.path
IM_CONFIG_PHASE=1
COMPIZ_CONFIG_PROFILE=ubuntu
GDMSESSION=ubuntu
SESSIONTYPE=gnome-session
GTK2_MODULES=overlay-scrollbar
XDG_SEAT=seat0
HOME=/root
SHLVL=1
GNOME_DESKTOP_SESSION_ID=this-is-deprecated
UPSTART_INSTANCE=
UPSTART_EVENTS=xsession started
XDG_SESSION_DESKTOP=ubuntu
LOGNAME=root
COMPIZ_BIN_PATH=/usr/bin/
QT4_IM_MODULE=xim
XDG_DATA_DIRS=/usr/share/ubuntu:/usr/share/gnome:/usr/local/share/:/usr/share/
DBUS_SESSION_BUS_ADDRESS=unix:abstract=/tmp/dbus-y6EgZti58V
LESSOPEN=| /usr/bin/lesspipe %s
INSTANCE=
UPSTART_JOB=unity7
XDG_RUNTIME_DIR=/run/user/0
DISPLAY=:0
XDG_CURRENT_DESKTOP=Unity
GTK_IM_MODULE=ibus
LESSCLOSE=/usr/bin/lesspipe %s %s
XAUTHORITY=/root/.Xauthority
_=/usr/bin/env
```

在Container中执行env命令的结果：

```text
root@33b328a4095b:/go/src/github.com/docker/docker
RSRC_COMMIT=ba14da1f827188454a4591717fff29999010887f
HOSTNAME=33b328a4095b
TERM=xterm
REGISTRY_COMMIT=47a064d4195a9b56133891bbb13620c3ac83a827
SECCOMP_VERSION=2.2.3
OSX_SDK=MacOSX10.11.sdk
OSX_CROSS_COMMIT=8aa9b71a394905e6c5f4b59e2b97b87a004658a4
REGISTRY_COMMIT_SCHEMA1=ec87e9b6971d831f0eff752ddb54fb64693e51cd
LVM2_VERSION=2.02.103
DOCKER_CROSSPLATFORMS=linux/386 linux/arm   darwin/amd64    freebsd/amd64 freebsd/386 freebsd/arm   windows/amd64 windows/386
PATH=/go/bin:/usr/local/go/bin:/osxcross/target/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
NOTARY_VERSION=v0.2.0
GOARM=5
PWD=/go/src/github.com/docker/docker
GO_TOOLS_COMMIT=823804e1ae08dbb14eb807afc7db9993bc9e3cc3
TOMLV_COMMIT=9baf8a8a9f2ed20a8e54160840c492f937eeaf9a
HOME=/root
SHLVL=1
DOCKER_PY_COMMIT=e2878cbcc3a7eef99917adc1be252800b0e41ece
GO_LINT_COMMIT=32a87160691b3c96046c0c678fe57c5bef761456
DOCKER_BUILDTAGS=apparmor seccomp selinux
GOPATH=/go:/go/src/github.com/docker/docker/vendor
container=docker
GO_VERSION=1.6
_=/usr/bin/env
```

## 第5章 Dropped capabilities方法 <a id="&#x7B2C;5&#x7AE0;-dropped-capabilities&#x65B9;&#x6CD5;"></a>

帖子上提到了：”Otherwise, if you are root, you can try to perform mknod or mount operation, if it fails, you are most likely in a container with dropped capabilities.”

## 第6章 Mount方法（原创） <a id="&#x7B2C;6&#x7AE0;-mount&#x65B9;&#x6CD5;&#x539F;&#x521B;"></a>

在Host和Container中执行mount命令的结果是不同的，可以利用这一点进行判断。

在Host中执行mount命令的结果：

```text
root@ubuntu:~# mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
udev on /dev type devtmpfs (rw,nosuid,relatime,size=1997320k,nr_inodes=499330,mode=755)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,noexec,relatime,size=402932k,mode=755)
/dev/sda1 on / type ext4 (rw,relatime,errors=remount-ro,data=ordered)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)
tmpfs on /run/lock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=5120k)
tmpfs on /sys/fs/cgroup type tmpfs (rw,mode=755)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/lib/systemd/systemd-cgroups-agent,name=systemd)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_cls,net_prio)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event,release_agent=/run/cgmanager/agents/cgm-release-agent.perf_event)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpu,cpuacct)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset,clone_children)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb,release_agent=/run/cgmanager/agents/cgm-release-agent.hugetlb)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=27,pgrp=1,timeout=0,minproto=5,maxproto=5,direct)
mqueue on /dev/mqueue type mqueue (rw,relatime)
debugfs on /sys/kernel/debug type debugfs (rw,relatime)
vmware-vmblock on /run/vmblock-fuse type fuse.vmware-vmblock (rw,relatime,user_id=0,group_id=0,default_permissions,allow_other)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime)
fusectl on /sys/fs/fuse/connections type fusectl (rw,relatime)
cgmfs on /run/cgmanager/fs type tmpfs (rw,relatime,size=100k,mode=755)
tmpfs on /run/user/0 type tmpfs (rw,nosuid,nodev,relatime,size=402932k,mode=700)
gvfsd-fuse on /run/user/0/gvfs type fuse.gvfsd-fuse (rw,nosuid,nodev,relatime,user_id=0,group_id=0)
gvfsd-fuse on /run/user/0/gvfs type fuse.gvfsd-fuse (rw,nosuid,nodev,relatime,user_id=0,group_id=0)
binfmt_misc on /proc/sys/fs/binfmt_misc type binfmt_misc (rw,relatime)
tracefs on /sys/kernel/debug/tracing type tracefs (rw,relatime)
```

在Container中执行mount命令的结果：

```text
root@33b328a4095b:/go/src/github.com/docker/docker# mount
none on / type aufs (rw,relatime,si=f93f2580cb95eee4,dio,dirperm1)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev type tmpfs (rw,nosuid,mode=755)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=666)
mqueue on /dev/mqueue type mqueue (rw,nosuid,nodev,noexec,relatime)
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /sys/fs/cgroup type tmpfs (rw,nosuid,nodev,noexec,relatime,mode=755)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_cls,net_prio)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event,release_agent=/run/cgmanager/agents/cgm-release-agent.perf_event)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpu,cpuacct)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset,clone_children)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb,release_agent=/run/cgmanager/agents/cgm-release-agent.hugetlb)
/dev/sda1 on /var/lib/docker type ext4 (rw,relatime,errors=remount-ro,data=ordered)
/dev/sda1 on /etc/resolv.conf type ext4 (rw,relatime,errors=remount-ro,data=ordered)
/dev/sda1 on /etc/hostname type ext4 (rw,relatime,errors=remount-ro,data=ordered)
/dev/sda1 on /etc/hosts type ext4 (rw,relatime,errors=remount-ro,data=ordered)
shm on /dev/shm type tmpfs (rw,nosuid,nodev,noexec,relatime,size=65536k)
devpts on /dev/console type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
none on /sys/kernel/security type securityfs (rw,relatime)
none on /tmp type tmpfs (rw,relatime)
```

## 第7章 设备文件方法（不成功） <a id="&#x7B2C;7&#x7AE0;-&#x8BBE;&#x5907;&#x6587;&#x4EF6;&#x65B9;&#x6CD5;&#x4E0D;&#x6210;&#x529F;"></a>

在Host和Container中执行ls /dev命令的结果是不同的，可以利用这一点进行判断。

但是目前还没发现本质的不同点。

## 第8章 文件系统方法 <a id="&#x7B2C;8&#x7AE0;-&#x6587;&#x4EF6;&#x7CFB;&#x7EDF;&#x65B9;&#x6CD5;&#x539F;&#x521B;"></a>

在Host和Container中执行df -h命令的结果是不同的，可以利用这一点进行判断。

在Host中执行df -h命令的结果：

```text
root@ubuntu:~# df -h
Filesystem      Size  Used Avail Use
udev            2.0G     0  2.0G   0
tmpfs           394M   12M  383M   3
/dev/sda1        36G  9.7G   24G  29
tmpfs           2.0G  928K  2.0G   1
tmpfs           5.0M  4.0K  5.0M   1
tmpfs           2.0G     0  2.0G   0
cgmfs           100K     0  100K   0
tmpfs           394M   52K  394M   1
```

其中，cgmanager是cgroup的管理器daemon，运行于root下。

在Container中执行df -h命令的结果：

```text
root@8a601702e06f:/go/src/github.com/docker/docker# df -h
Filesystem      Size  Used Avail Use
none             36G  9.7G   24G  29
tmpfs           2.0G     0  2.0G   0
tmpfs           2.0G     0  2.0G   0
/dev/sda1        36G  9.7G   24G  29
shm              64M     0   64M   0
none            2.0G     0  2.0G   0
```

