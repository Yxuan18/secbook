# 推荐二

## Docker daemon 配置

本节列出了修改、增强Docker daemon\(服务器端\)行为的推荐基线。以下的设置会影响所有容器。

**注意：Docker daemon 的可选配置可以使用Debian和Ubuntu下的/etc/sysconfig/docker或/etc/default/docker文件来进行配置。同时，docker daemon 模式与 `/user/bin/dockerd`，`docker` 服务的 `-d`或`daemon`参数等价。**

### 2.1 限制容器间\(Scored\)

**配置适用性**

* **Level 1- Docker**

**描述**

默认情况下，在同一主机上的容器是允许互相之间通信的。如果不想，限制所有容器间的通信。在需要容器间通讯时，将相关的容器 link 起来。

**缘由**

默认情况下，在同一主机上的所有容器之间使用不受限制的网络通信。 因此，每个容器都有可能读取同一主机容器网络上的所有数据包。 这可能会导致意外和不必要的将信息泄露给其他容器。 因此，限制容器间的通信。

**审计**

```text
ps -ef | grep dockerd
```

确保 `'--icc'`参数被设置为 `'false'`，或者通过如下命令确认。

```text
docker network ls --quiet | xargs xargs docker network inspect --format '{{ .Name }}:{{ .Options }} '
```

将会返回：`com.docker.network.bridge.enable_icc:false`。

**修正**

以`'--icc=false'` 参数运行docker daemon。

例如：

```text
/usr/bin/dockerd --icc=false
```

**影响**

容器间通信将被禁用。 容器将不能与同一主机上的另一个容器进行通信。 如果需要在同一主机上的容器之间进行通信，则需要使用容器链接来显式定义它。

**默认值**

默认情况下，是允许容器间通信的。

**参考文献**

```text
1. https://docs.docker.com/articles/networking
```

### 2.2 设置logging级别\(Scored\)

**配置适用性**

* **Level 1- Docker**

**描述**

设置Docker daemon的日志级别为 `'info'`

**缘由**

设置适当的日志级别，配置Docker daemon 以记录您稍后需要查看的事件。将捕获除调试日志以外，`“info”`及其以上的基本日志级别的所有日志。 除非需要，否则不应在`“debug”`日志级别运行Docker daemon。

**审计**

```text
ps -ef | grep dockerd
```

确保没有`'--log-level'`参数，如果有，修改为`'info'`级别。

**修正**

以如下方式运行 docker daemon：

```text
dockerd --log-level="info"
```

**影响**

无

**默认值**

默认情况下，Docker daemon 的日志级别为 `'info'`。

**参考文献**

```text
1. https://docs.docker.com/engine/reference/commandline/daemon/
```

### 2.3 允许Docker改变iptables\(Scored\)

**配置适用性**

* **Level 1- Docker**

**描述**

`Iptables`是用于设置，维护和检查Linux内核中的IP包过滤规则表。 允许Docker daemon 更改`iptables`。

**缘由**

如果您选择这么做，Docker也不会更改您系统的 `iptables`。 Docker 服务会自动根据您如何选择容器的网络选项（如果允许的话）对`iptables`进行必要的修改。 建议让Docker 服务自动更改 `iptables`，以避免可能妨碍容器与外界通信的网络配置错误。 此外，每次选择运行容器或修改网络选项时，都可以减少更新 `iptables` 的麻烦。

**审计**

```text
ps -ef | grep dockerd
```

确保`“—iptables”`参数不存在或未设置为`“false”`。

**修正**

不要使用 `“--iptables = false”`参数运行Docker daemon。

例如，不要以如下方式启动Docker daemon：

```text
dockerd --iptables=false
```

**影响**

无

**默认值**

默认情况下，`“iptables”`设置为`“true”`。

**参考文献**

```text
1. https://docs.docker.com/v1.8/articles/networking/
```

### 2.4 不使用不安全的镜像仓库\(Scored\)

**配置适用性**

* **Level 1- Docker**

**描述**

Docker认为私有仓库是安全的或不安全的。 默认情况下，仓库被认为是安全的。

**缘由**

一个安全的仓库使用TLS。 仓库的CA证书副本放置在Docker主机的`'/etc/docker/certs.d/<registry-name>/'`目录中。 不安全的仓库是没有有效的仓库证书或没有使用TLS。 您不应该在生产环境中使用任何不安全的仓库。 不安全的注册仓库可能会被篡改，从而导致生产系统受到影响。 此外，如果一个仓库被标记为不安全，那么`'docker pull'`, `'docker push'`和`'docker search'`命令将不会产生错误消息，用户可能会无限期地使用不安全的仓库，而不会得知潜在的危险。

**审计**

运行`docker info`或者执行下面的命令来查看是否使用了不安全的仓库：

```text
ps -ef | grep dockerd
```

确保`'--insecure-registry'`参数不存在。

**修正**

不要使用任何不安全的仓库。 例如，不要以如下方式启动Docker daemon：

```text
dockerd --insecure-registry 10.1.0.0/16
```

**影响**

无

**默认值**

默认情况下，Docker假定所有本地的仓库都是安全的。

**参考文献**

```text
1. https://docs.docker.com/registry/insecure/
```

### 2.5 不要使用aufs存储驱动\(Scored\)

**配置适用性**

* **Level 1- Docker**

**描述**

不要使用`'aufs'`作为你的Docker实例的存储驱动。

**缘由**

`'aufs'` 存储驱动程序是最旧的存储驱动程序。 它基于Linux内核补丁集，不太可能被合并到主要的Linux内核中。 `“aufs”`驱动程序会导致一些严重的内核崩溃。 `'aufs'`只有Docker的传统支持。 最重要的是，在许多使用了最新Linux内核的Linux发行版中，`“aufs”`不再是受支持的驱动程序。

**审计**

执行以下命令并确认`“aufs”` 不被用作存储驱动程序：

```text
docker info | grep -e "^Storage Driver:\s*aufs\s*$"
```

上面的命令不应该返回任何东西。

**修正**

不要明确地使用`“aufs”`作为存储驱动程序。 例如，不要以如下方式启动Docker守护进程：

```text
dockerd --storage-driver aufs
```

**影响**

`'aufs'`是唯一允许容器共享可执行文件和共享库内存的存储驱动程序。 如果您使用相同的程序或库运行数千个容器，这可能会很有用。

**默认值**

默认情况下，Docker在大多数平台上使用 `“devicemapper”`作为存储驱动程序。 默认存储驱动程序可以根据您的操作系统供应商而有所不同，您应该使用您的供应商支持的首选存储驱动程序。

**参考文献**

```text
1. http://docs.docker.com/reference/commandline/cli/#daemon-storage-driver- option
2. http://muehe.org/posts/switching-docker-from-aufs-to-devicemapper/
3. http://jpetazzo.github.io/assets/2015-03-05-deep-dive-into-docker-storage-
drivers.html#1
4. https://docs.docker.com/engine/userguide/storagedriver/
```

### 2.6 为Docker daemon 配置TLS认证\(Scored\)

**配置适用性**

* **Level 1- Docker**

**描述**

可以让Docker daemon 监听特定的IP和端口以及任何其他Unix 非默认套接字。 配置TLS身份验证，以限制访问Docker daemon 的IP和端口。

**缘由**

默认情况下，Docker daemon绑定到非网络连接的Unix套接字并以 `“root”`权限运行。 如果将默认docker daemon绑定到TCP端口或任何其他Unix套接字，那么任何有权访问该端口或套接字的人都可以完全访问Docker守护进程，并转而访问主机系统。 因此，不应该将Docker daemon绑定到另一个IP /端口或Unix套接字。 如果您必须通过网络套接字公开Docker daemon，请为daemon 和Docker Swarm API（如果使用）配置TLS身份验证。 这将限制通过网络到Docker daemon的连接，将通过TLS成功进行身份验证的客户端限制在一定数量。

**审计**

```text
ps -ef | grep dockerd
```

确保以下参数存在：

* `'—tlsverify'`
* `'—tlscacert'`
* `'—tlscert'`
* `'—tlskey'`

**修正**

遵循Docker文档或其他参考中提到的步骤。

**影响**

您需要管理和保护Docker daemon和Docker客户端的证书和密钥。

**默认值**

默认情况下，没有配置TLS认证。

**参考文献**

```text
1. http://docs.docker.com/articles/https/
```

### 2.7根据需要设置默认的ulimit（Not Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

在您的环境中根据需要设置默认的ulimit选项。

**缘由**

`ulimit` 提供了对shell可用的资源以及由它启动的进程的控制。 合理地设置系统资源限制可以使您免受许多灾难的困扰，例如，fork 炸弹。 有时候，即使友好的用户和合法的进程也会过度使用系统资源，从而导致系统无法使用。 设置Docker daemon的默认`ulimit`将强制所有容器实例的ulimit。 你不需要为每个容器实例设置 `ulimit`。 但是，如果需要，默认的 `ulimit`可以在容器运行时重写。 因此，要控制系统资源，请在您的环境中根据需要定义默认的`ulimit`。

**审计**

```text
ps -ef | grep dockerd
```

确保`“--default-ulimit”`参数被设置为合适的值。

**修正**

在daemon 模式下运行docker，并在您的环境中根据相应的ulimits传递`“--default-ulimit”`作为参数。

例如，

```text
dockerd --default-ulimit nproc=1024:2408 --default-ulimit nofile=100:200
```

**影响**

如果 `ulimits`设置不正确，可能无法实现所需的资源控制，甚至可能导致系统无法使用。

**默认值**

默认情况下，没有设置ulimit。

**参考文献**

```text
1. https://docs.docker.com/engine/reference/commandline/daemon/#default- ulimits
```

### 2.8启用用户名称空间支持（Scored）

**配置适用性**

* **Level 2- Docker**

**描述**

在Docker daemon中启用用户名称空间以利用容器用户与主机用户的重新映射。 如果您正在使用的容器没有在容器镜像中定义明确的容器用户，则此建议是有益的。 如果您正在使用的容器镜像具有预定义的非root用户，则可以跳过此建议，因为此功能仍处于初始阶段，可能会给您带来不可预知的问题和复杂性。

**缘由**

Docker daemon中的Linux内核用户名称空间为Docker主机系统提供了额外的安全性。 它允许容器具有，在主机系统使用的传统用户和组范围之外的，唯一范围的用户和组ID。 例如，`root` 用户将在容器内具有预期的管理权限，但可以有效地映射到主机系统上的非特权UID。

**审计**

```text
ps -p $(docker inspect --format='{{ .State.Pid }}' <CONTAINER ID>) -o pid,user
```

以上命令将查找容器的PID，然后列出与容器进程关联的主机用户。 如果容器进程以root用户身份运行，不符合此基线。 或者，您可以运行 `docker info` 以确保在`Security Options`下列出`userns`：

```text
docker info --format '{{ .SecurityOptions }}'
```

**修正**

请参阅Docker文档，了解可以根据您的要求配置的各种方法。 您的步骤也可能因平台而异 。例如，在Red Hat上，sub-UID和 sub-GID映射创建不能自动工作。 您可能需要创建自己的映射。

但是，高层次步骤如下：

步骤1：确保文件`/ etc / subuid`和`/ etc / subgid`存在。

```text
touch /etc/subuid /etc/subgid
```

步骤2：用`--userns-remap`标志启动docker daemon。

```text
dockerd --userns-remap=default
```

**影响**

用户命名空间重新映射使得不少Docker功能不兼容，而且目前也打破了一些功能。 查看Docker文档和参考链接了解详细信息。

**默认值**

默认情况下，用户命名空间不会被重新映射。

**参考文献**

```text
1. http://man7.org/linux/man-pages/man7/user_namespaces.7.html
2. https://docs.docker.com/engine/reference/commandline/daemon/
3. http://events.linuxfoundation.org/sites/events/files/slides/User%20Namespaces
%20-%20ContainerCon%202015%20-%2016-9-final_0.pdf
4. https://github.com/docker/docker/issues/21050
```

### 2.9 确认默认cgroup使用情况（Scored）

**配置适用性**

* **Level 2- Docker**

**描述**

`--cgroup-parent`选项允许您为所有容器设置默认的cgroup父项。 如果没有特定用例，则应将此设置保留为默认值。

**缘由**

系统管理员通常定义容器应该在哪个`cgroup`下运行。 即使 cgroups没有被系统管理员明确定义，默认情况下，容器会在 `docker` cgroup下运行。 这是可能的附加到一个不同于默认的cgroup。 这个用法应该被监视和确认。 通过附加到不同于默认的cgroup，可以不均匀地共享资源，从而可能导致主机资源匮乏。

**审计**

```text
ps -ef | grep dockerd
```

确保 `“--cgroup-parent”`参数未设置，或者设置为非默认cgroup。

**修正**

默认设置足够好，可以保持原样。 如果你想专门设置一个非缺省的cgroup，在启动时将`--cgroup-parent`参数传递给docker daemon。 例如，

```text
dockerd --cgroup-parent=/foobar
```

**影响**

无

**默认值**

默认情况下，docker daemon为 fs cgroup驱动程序使用 `/docker`，为 systemd cgroup驱动程序使用 `system.slice`。

**参考文献**

```text
1. https://docs.docker.com/engine/reference/commandline/daemon/
```

### 2.10 在需要时才改变基础设备大小（Scored）

**配置适用性**

* **Level 2- Docker**

**描述**

在某些情况下，您可能需要容量大于10G的容器。 在这些情况下，请仔细选择基础设备大小。

**缘由**

在 daemon 重启时，可以增加基本设备的大小。 增加基础设备尺寸允许所有将来的镜像和容器成为新的基础设备尺寸。 用户可以使用此选项来扩展基础设备尺寸，但不允许收缩。 该值影响可能已经被初始化并被拉取的镜像继承的系统范围的“基本”空文件系统。 虽然如果文件系统为空，文件系统不会分配增加的大小，但根据设备的大小，它将占用更多的空间。 这可能会导致文件系统被过度分配或已满而导致拒绝服务。

**审计**

```text
ps -ef | grep dockerd
```

执行上面的命令，它不应该显示任何`--storage-opt dm.basesize`参数。

**修正**

直到需要时，才设置 `--storage-opt dm.basesize`。

**影响**

无

**默认值**

默认的基础设备大小是10G。

**参考文献**

```text
1. https://docs.docker.com/engine/reference/commandline/daemon/#storage- driver-options
```

### 2.11 使用授权插件（Scored）

**配置适用性**

* **Level 2- Docker**

**描述**

使用授权插件来管理对Docker daemon的访问。

**缘由**

Docker的开箱即用授权模式是全有或者全无。 任何有权访问Docker daemon的用户都可以运行任何Docker客户端命令。 对于使用Docker的远程API来调用 daemon 的调用者也是如此。 如果您需要更大的访问控制，您可以创建授权插件并将其添加到您的Docker daemon 配置。 使用授权插件，Docker管理员可以配置访问策略粒度来管理对Docker daemon的访问。

**审计**

```text
ps -ef | grep dockerd
```

确保 `“--authorization-plugin”`参数被设置为合适的值。

**修正**

步骤1：安装/创建授权插件； 步骤2：根据需要配置授权策略。 步骤3：如下启动docker daemon：

```text
dockerd --authorization-plugin=<PLUGIN_ID>
```

**影响**

每个docker命令专门通过授权插件机制。 这可能会导致性能下降。

**默认值**

默认情况下，授权插件没有设置。

**参考文献**

```text
1. https://docs.docker.com/engine/reference/commandline/daemon/#access- authorization
2. https://docs.docker.com/engine/extend/authorization/ 
3. https://github.com/twistlock/authz
```

### 2.12 配置集中和远程日志（Scored）

**配置适用性**

* **Level 2- Docker**

**描述**

Docker现在支持各种日志驱动程序。 存储日志的最佳方式是支持集中式和远程日志。

**缘由**

集中和远程日志确保所有重要的日志都是安全的，尽管发生灾难性事件。 Docker现在支持各种各样的日志驱动程序。 使用最适合你的环境的那个。

**审计**

运行 `docker info`并确保根据需要设置 `Logging Driver`属性。

```text
docker info --format '{{ .LoggingDriver }}'
```

或者，如果配置的话， 下面的命令会给你 `--log-driver`设置，确保它是适当的值。

```text
ps -ef | grep dockerd
```

**修正**

步骤1：按照其文档设置所需的日志驱动程序。

步骤2：使用该日志驱动程序启动docker守护进程。

例如，

```text
dockerd --log-driver=syslog --log-opt syslog-address=tcp://192.xxx.xxx.xxx
```

**影响**

无

**默认值**

默认情况下，容器日志被保存为json文件

**参考文献**

```text
1. https://docs.docker.com/engine/admin/logging/overview/
```

### 2.13 禁用旧版仓库的操作（v1）（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

最新的Docker 仓库是v2。 旧仓库版本（v1）上的所有操作都应受到限制。

**缘由**

Docker 仓库v2相比于v1，带来了许多性能和安全性方面的改进。 它支持容器镜像来源以及其他安全功能，如镜像签名和验证。 因此，Docker遗留仓库的操作应该受到限制。

**审计**

```text
ps -ef | grep dockerd
```

上面的命令应该列出`--disable-legacy-registry` 传递给docker守护进程的选项。

**修正**

以如下方式启动docker daemon：

```text
dockerd --disable-legacy-registry
```

**影响**

旧版仓库的操作将受到限制。

**默认值**

默认情况下，旧版仓库操作是允许的。

**参考文献**

```text
1. https://docs.docker.com/engine/reference/commandline/daemon/
2. https://github.com/docker/docker/issues/8093
3. https://github.com/docker/docker/issues/9015
4. https://github.com/docker/docker-registry/issues/612
5. https://docs.docker.com/registry/spec/api/
6. https://the.binbashtheory.com/creating-private-docker-registry-2-0-with-token-
authentication-service/
7. https://blog.docker.com/2015/07/new-tool-v1-registry-docker-trusted-registry-
v2-open-source/
8. http://www.slideshare.net/Docker/docker-registry-v2
```

### 2.14 启用实时还原（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

`'--live-restore'`可以在docker中完全支持 daemon-less 的容器。 它确保docker不会在关闭或恢复时停止容器，并在重新启动时正确重新连接到容器。

**缘由**

其中一个重要的安全特性是可用性。 在docker daemon中设置`'--live-restore'`标志可以确保当docker daemon 不可用时容器的执行不会中断。 这也意味着现在可以在不停机的情况下，更容易更新和修补docker daemon 。

**审计**

运行 `Docker info` 和确保`Live Restore Enabled` 属性设置为 `true`。

```text
docker info --format '{{ .LiveRestoreEnabled }}'
```

或者运行下面的命令，并确保使用 `--live-restore`。

```text
ps -ef | grep dockerd
```

**修正**

以 `'--live-restore'` 运行docker daemon。

例如，

```text
dockerd --live-restore
```

**影响**

无

**默认值**

默认情况下，没有启用`--live-restore` 。

**参考文献**

```text
1. https://github.com/docker/docker/pull/23213
```

### 2.15 在不需要swarm 时，不要启用它（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

除非需要，否则不要在Docker引擎实例上启用 swarm 模式。

**缘由**

默认情况下，Docker引擎实例不会侦听任何网络端口，所有与客户端的通信都通过Unix套接字进行。 在Docker引擎实例上启用Docker swarm模式时，系统将打开多个网络端口，并将其提供给网络中的其他系统以用于群集管理和节点通信。 打开系统上的网络端口会增加攻击面，除非需要，否则应该避免这种情况。

**审计**

查看 `docker info`命令的输出。 如果输出包括 `Swarm：active`，则表示在Docker引擎上已经激活了 swarm 模式。 确认Docker引擎实例上是否需要swarm模式。

**修正**

如果 swarm 模式在系统上出错，请运行

`docker swarm leave`

**影响**

无

**默认值**

默认情况下，没有启用Docker swarm 模式。

**参考文献**

```text
1. https://docs.docker.com/engine/reference/commandline/swarm_init/
```

### 2.16 控制swarm 中管理节点的数量（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

确保 swarm 创建所需管理节点的最小数量。

**缘由**

swarm 中的管理节点可以控制swarm，并修改其以修改安全参数的配置。 拥有过多的管理节点会使swarm 更容易受到损害。 如果管理节点中不需要容错功能，则应当选择一个节点作为管理者。 如果需要容错，则应该配置最小的实际奇数以达到适当的容错水平。

**审计**

运行 `docker info`并验证管理节点的数量。

```text
docker info --format '{{ .Swarm.Managers }}'
```

或者运行下面的命令。

```text
docker node ls | grep 'Leader'
```

**修正**

如果配置的管理节点数量过多，则可以使用以下命令将多余的工人降级：

```text
docker node demote <ID>
```

其中 `<ID>`是要降级节点的ID值。

**影响**

无

**默认值**

一个管理节点就是启动一个给定集群所需要的一切。

**参考文献**

```text
1. https://docs.docker.com/engine/swarm/manage-nodes/
2. https://docs.docker.com/engine/swarm/admin_guide/#/add-manager-nodes-for-
fault-tolerance
```

### 2.17 将 swarm 服务绑定到特定的主机接口（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

默认情况下，Docker swarm 服务将侦听主机上的所有接口，这对于主机具有多个网络接口的 swarm 的操作可能不是必需的。

**缘由**

当swarm 初始化时，`--listen-addr`标志的缺省值是`0.0.0.0:2377`，这意味着swarm服务将监听主机上的所有接口。 如果主机具有多个网络接口，则这可能是不合需要的，因为它可能将 docker swarm服务暴露给不涉及swarm 操作的网络。 通过将特定的IP地址传递给 `--listen-addr`，可以指定一个特定的网络接口来限制这种暴露。

**审计**

在端口2377 / TCP上列出网络监听器（docker swarm的默认设置），并确认它只监听特定的接口。 例如，使用ubuntu可以使用以下命令完成：

```text
netstat -lt | grep -i 2377
```

**修正**

对此进行修复需要重新初始化swarm ，以指定`--listen-addr`参数的特定接口。

**影响**

无

**默认值**

默认情况下，docker swarm服务监听所有可用的主机接口。

**参考文献**

```text
1. https://docs.docker.com/engine/reference/commandline/swarm_init/#/listen- addr-value
2. https://docs.docker.com/engine/swarm/admin_guide/#/recover-from-disaster
```

**注意：我注意到看这个的几个观点。 docker swarm update似乎没有更改listen-addr参数。 对于补救措施，我确实想知道是否可以使用 -- force-new-swarm 改变这种情况，但是我不确定对swarm 会有什么其他的影响，因此只剩下一般要求来重新初始化swarm。** **另外有趣的是，在7946 / TCP上运行的节点通信服务并不遵守--listen-addr参数。 这对我来说似乎是一个错误，我可能会在github上进行更多的探索之后提出一个问题。**

### 2.18 禁用用户级代理（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

docker daemon在端口暴露时启动用于端口转发的用户级代理服务。 在 hairpin NAT可用的地方，这种服务通常是多余的，可以被禁用。

**缘由**

Docker引擎提供了两种将主机端口转发到容器—hairpin NAT和用户级代理—机制。 在大多数情况下，hairpin NAT模式是首选，因为它提高了性能，并使用本地Linux iptables功能，而不是一个额外的组件。 在hairpin NAT可用的情况下，应在启动时禁用用户级代理，以减少安装的攻击面。

**审计**

```text
ps -ef | grep dockerd
```

确保 `--userland-proxy`参数设置为 `false`。

**修正**

以如下方式运行Docker daemon：

```text
dockerd --userland-proxy=false
```

**影响**

某些带有较旧Linux内核的系统可能无法支持 hairpin NAT，因此需要用户级代理服务。 此外，一些网络设置可能会受到删除用户级代理的影响。

**默认值**

默认情况下，启用了用户级代理。

**参考文献**

```text
1. http://windsock.io/the-docker-proxy/
2. https://github.com/docker/docker/issues/14856
3. https://github.com/docker/docker/issues/22741
4. https://docs.docker.com/engine/userguide/networking/default_network/binding/
```

### 2.19 加密overlay 网络不同节点上的容器之间交换的数据（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

加密overlay 网络不同节点上的容器之间交换的数据。

**缘由**

默认情况下，overlay 网络不同节点上的容器之间交换的数据不会被加密。 这可能会暴露容器节点之间的流量。

**审计**

运行以下命令并确保每个overlay 网络已被加密。

```text
docker network ls --filter driver=overlay --quiet | xargs docker network inspect --
format '{{.Name}} {{ .Options }}'
```

**修正**

使用 `--opt encrypted`标志创建 overlay 网络。

**影响**

无

**默认值**

默认情况下， overlay 网络不同节点上的容器之间交换的数据在Docker swarm模式下不进行加密。

**参考文献**

```text
1. https://docs.docker.com/engine/userguide/networking/overlay-security-model/ 
2. https://github.com/docker/docker/issues/24253
```

### 2.20 如果需要，应用daemon-wide范围的自定义seccomp配置文件（Not Scored）

**配置适用性**

* **Level 2- Docker**

**描述**

如果需要，您可以选择在 daemon-wide 级别使用自定义的seccomp配置文件，用以覆盖Docker默认的seccomp配置文件。

**缘由**

大量的系统调用暴露于每个用户级进程，其中许多系统调用在整个生命周期中都未被使用。 大多数应用程序不需要所有的系统调用，因此可以通过减少可用的系统调用来获益。 减少的一组系统调用减少了暴露给应用程序的内核面，从而临时提升应用程序安全性。 您可以应用您自己自定义的seccomp配置文件，而不是Docker的默认seccomp配置文件。 或者，如果Docker的默认配置文件适合您的环境，则可以选择忽略此建议。

**审计**

运行以下命令并查看`“Security Options”`部分列出的seccomp配置文件。 如果它是默认的，那就意味着，应用Docker的默认seccomp配置文件。

```text
docker info --format '{{ .SecurityOptions }}'
```

**修正**

默认情况下，应用Docker的默认seccomp配置文件。 如果这适用于您的环境，则不需要采取任何行动。 或者，如果您选择应用自己的seccomp配置文件，请在daemon 启动时使用 `--seccomp-profile`标志，或将其放在daemon 运行时参数文件中。

```text
dockerd --seccomp-profile </path/to/seccomp/profile>
```

**影响**

错误配置的seccomp配置文件可能会中断您的容器环境。 Docker默认的阻塞调用已经仔细审查。 这些解决了容器环境中的一些关键漏洞/问题（例如，内核密钥环调用）。 所以，在重写默认值时，您应该非常小心。

**默认值**

默认情况下，Docker使用seccomp配置文件。

**参考文献**

```text
1. https://github.com/docker/docker/pull/26276
```

### 2.21 避免生产中的实验性特征（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

避免生产中的使用实验性功能。

**缘由**

实验现在是一个docker daemon运行时的标志，而不需单独构建。`— experimental` 作为运行时标志传递给docker daemon，激活实验性功能。 实验现在被认为是一个稳定的版本，但有一些功能可能没有经过测试，不能保证API的稳定性。

**审计**

运行以下命令并确保在`“Server"`部分中的`“Experimental”`属性设置为 `“false”`。

```text
docker version --format '{{ .Server.Experimental }}'
```

**修正**

不要将`--experimental`作为运行时参数传递给docker daemon。

**影响**

无

**默认值**

默认情况下，docker daemon不会激活实验性功能。

**参考文献**

```text
1. https://github.com/docker/docker/issues/26713 
2. https://github.com/docker/docker/pull/27223
```

### 2.22 使用Docker的 secret 管理命令来管理Swarm集群中的seret（Not Scored）

**配置适用性**

* **Level 2- Docker**

**描述**

使用Docker的内置 `secret` 管理命令。

**缘由**

Docker有各种命令来管理Swarm集群中的secrets。 这是Docker 支持未来secret的基础，可能会有改进，如支持Windows，支持不同的存储等。

**审计**

如果可以，在swarm 管理节点上，运行以下命令，并确保在您的环境中使用了 `docker secret`：

```text
docker secret ls
```

**修正**

按照 `docker secret` 文档，使用它来有效地管理 secret。

**影响**

无

**默认值**

不适用

**参考文献**

```text
1. https://github.com/docker/docker/pull/27794
```

### 2.23 在自动锁定模式下运行swarm 管理节点（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

在自动锁定模式下运行Docker swarm 管理节点。

**缘由**

当Docker重新启动时，用于加密 swarm 节点之间通信的TLS密钥以及用于加密和解密磁盘上Raft日志的密钥都被加载到每个管理节点的内存中。 您应该保护相互的TLS加密密钥和用于加密和解密Raft日志的密钥。 这个保护可以通过使用`—autolock`标志来初始化swarm来启用。 启用`--autolock`后，当Docker重新启动时，您必须先使用Docker在swarm初始化时生成的加密密钥解锁swarm。

**审计**

运行下面的命令。 如果输出密钥，则意味着swarm已经用`--autolock`标志初始化了。 如果输出 `no unlock key is set`，则意味着swarm 未使用`—autolock`标志进行初始化，不符合此基线。

```text
docker swarm unlock-key
```

**修正**

如果您正在初始化swarm，使用下面的命令。

```text
docker swarm init --autolock
```

如果要在现有swarm管理节点上设置`--autolock`，请使用以下命令。

```text
docker swarm update --autolock
```

**影响**

在没有用户的手动干预输入解锁密钥，自动锁定模式下的swarm 不会重新启动恢复。 在某些部署中，这可能不利于可用性。

**默认值**

默认情况下，swarm 管理节点不在自动锁定模式运行。

**参考文献**

```text
1. https://github.com/mstanleyjones/docker.github.io/blob/af7dfdba8504f9b102fb3 1a78cd08a06c33a8975/engine/swarm/swarm_manager_locking.md
```

### 2.24 定期轮换swarm管理节点的自动锁定秘钥（Not Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

定期轮换swarm 管理节点的自动锁定密钥。

**缘由**

Swarm管理节点的自动锁定秘钥不会自动轮换。 作为最佳做法，您应该定期轮换它们。

**审计**

目前，没有机制来找出密钥在swarm管理节点上最后一次轮换的时间。 如果有密钥轮转记录，并且按预定义的频率轮换密钥，则应该与系统管理员核对。

**修正**

运行下面的命令来轮换秘钥。

```text
docker swarm unlock-key --rotate
```

此外，为了便于审计，维护密钥轮换记录并确保为密钥轮换建立预定义的频率。

**影响**

无

**默认值**

默认情况下，秘钥不会自动轮换。

**参考文献**

```text
1. https://github.com/mstanleyjones/docker.github.io/blob/af7dfdba8504f9b102fb3 1a78cd08a06c33a8975/engine/swarm/swarm_manager_locking.md
```

