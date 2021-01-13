# 推荐五

## 容器运行时

容器的启动方式对安全性有很大的影响。 可能提供潜在危险的运行时参数，这些参数可能会危及主机和主机上的其他容器。 验证容器运行时非常重要。 评估容器运行时的各种基线如下：

### 5.1 不要禁用AppArmor配置文件（Scored）- 此条有变化

**配置适用性**

* **Level 1- Docker**

**描述**

AppArmor是一个有效且易于使用的Linux应用程序安全系统。 它在很多Linux发行版上如Debian和Ubuntu都默认可用。

**缘由**

AppArmor通过执行被称为AppArmor配置文件的安全策略来保护Linux操作系统和应用程序免受各种威胁。 您可以为容器创建自己的AppArmor配置文件，也可以使用Docker的默认AppArmor配置文件。 这将在配置文件中定义的容器上执行安全策略。

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: AppArmorProfile={{.AppArmorProfile }}'
```

上述命令应该为每个容器实例返回一个有效的AppArmor配置文件。

**修正**

如果您的Linux操作系统可以使用AppArmor，请使用它。 您可能需要按照以下步骤进行操作：

1. 验证AppArmor是否已安装。 如果没有，请安装它。
2. 为Docker容器创建或导入AppArmor配置文件。
3. 将此配置文件置于强制模式。
4. 使用定制的AppArmor配置文件启动您的Docker容器。 例如，

```text
docker run --interactive --tty --security-opt="apparmor:PROFILENAME" centos /bin/bash
```

或者，您可以保留doker的默认apparmor配置文件

**影响**

容器（进程）将具有AppArmor配置文件中定义的一组限制。 如果您的AppArmor配置文件配置错误，则容器可能无法按预期完成工作。

**默认值**

默认情况下，将使用位于`/etc/apparmor.d/docker`的 `docker-default` AppArmor配置文件运行容器。

**参考文献**

```text
1. https://docs.docker.com/engine/security/apparmor/
2. http://docs.docker.com/articles/security/#other-kernel-security-features
3. https://github.com/docker/docker/blob/master/docs/security/apparmor.md 
4. http://docs.docker.com/reference/run/#security-configuration
```

### 5.2 如果适用，验证SELinux安全选项（Scored）

**配置适用性**

* **Level 2- Docker**

**描述**

SELinux是一个高效且易于使用的Linux应用程序安全系统。 它在默认情况下可用于很多Linux发行版，例如Red Hat和Fedora。

**缘由**

SELinux提供了强制访问控制（MAC）系统，大大增强了默认的自主访问控制（DAC）模型。 如果适用，您可以在Linux主机上启用SELinux，从而增加额外的安全性。

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: SecurityOpt={{.HostConfig.SecurityOpt }}'
```

上述命令应该返回为当前容器配置的所有安全选项。

**修正**

如果SElinux适用于您的Linux操作系统，请按照以下步骤设：

1. 设置SElinux状态
2. 设置 SElinux策略
3. 为docker容器创建或者导入一个SElinux策略模板
4. 启用SElinux选项，启动docker daemon，例如，

   ```text
   docker daemon --selinux-enabled
   ```

5. 使用安全选项启动docker容器，例如，

   ```text
   docker run --interactive --tty --security-opt label=level:TopSecret centos
   /bin/bash
   ```

**影响**

容器（进程）将具有SELinux策略中定义的一组限制。 如果您的SELinux策略配置错误，那么容器可能无法按预期完全工作。

**默认值**

默认情况下，不在容器上应用SELinux安全选项。

**参考文献**

```text
1. http://docs.docker.com/articles/security/#other-kernel-security-features
2. http://docs.docker.com/reference/run/#security-configuration
3. http://docs.fedoraproject.org/en-US/Fedora/13/html/Security-Enhanced_Linux/
```

### 5.3 限制容器内的Linux内核功能（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

默认情况下，Docker将启动一组受限制的Linux内核功能。 这意味着任何进程都可能被授予所需的功能，而不是`root`访问权限。 使用Linux内核功能，进程不必以超级用户身份运行几乎所有需要root权限的特定区域。

**缘由**

Docker支持添加和删除功能，允许使用非默认配置文件。 这可能会使Docker通过删除功能更安全，或者通过增加功能来降低Docker的安全性。 因此，建议删除除您的容器过程明确要求的所有功能。 例如，容器过程通常不需要下面的功能：

```text
NET_ADMIN
SYS_ADMIN
SYS_MODULE
```

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: CapAdd={{.HostConfig.CapAdd }} CapDrop={{ .HostConfig.CapDrop }}'
```

验证添加和删除的Linux内核功能是否与每个容器实例的容器进程所需的功能一致。

**修正**

执行以下命令添加所需的功能：

```text
$> docker run --cap-add={"Capability 1","Capability 2"} <Run arguments> <Container Image Name or ID> <Command>
```

例如，

```text
docker run --interactive --tty --cap-add={"NET_ADMIN","SYS_ADMIN"} centos:latest
/bin/bash
```

执行以下命令删除不需要的功能：

```text
$> docker run --cap-drop={"Capability 1","Capability 2"} <Run arguments> <Container Image Name or ID> <Command>
```

例如，

```text
docker run --interactive --tty --cap-drop={"SETUID","SETGID"} centos:latest /bin/bash
```

或者，您可以选择删除所有功能，只添加需要的功能：

```text
$> docker run --cap-drop=all --cap-add={"Capability 1","Capability 2"} <Run arguments> <Container Image Name or ID> <Command>
```

例如，

```text
docker run --interactive --tty --cap-drop=all --cap-add={"NET_ADMIN","SYS_ADMIN"}
centos:latest /bin/bash
```

**影响**

根据添加或删除哪些Linux内核功能，将适用于容器内的限制。

**默认值**

默认情况下，以下功能可用于容器：

```text
AUDIT_WRITE
CHOWN
DAC_OVERRIDE
FOWNER
FSETID
KILL
MKNOD
NET_BIND_SERVICE
NET_RAW
SETFCAP
SETGID
SETPCAP
SETUID
SYS_CHROOT
```

**参考文献**

```text
1. https://docs.docker.com/articles/security/#linux-kernel-capabilities
2. https://github.com/docker/docker/blob/master/daemon/execdriver/native/temp
late/default_template.go
3. http://man7.org/linux/man-pages/man7/capabilities.7.html
4. http://www.oreilly.com/webops-perf/free/files/docker-security.pdf
```

### 5.4 不要使用特权容器（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

使用`--privileged`标志会将所有Linux内核功能赋予容器，从而覆盖`--cap-add`和`--cap-drop`标志。 确保它不被使用。

**缘由**

`--privileged`标志赋予容器所有的功能，同时也提升了设备`cgroup`控制器强制执行的所有限制。 换句话说，容器可以做主机可以做的几乎所有事情。 这个标志存在允许特殊的用例，比如在Docker中运行Docker。

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: Privileged={{.HostConfig.Privileged }}'
```

以上命令应该为每个容器实例返回Privileged = false。

**修正**

不要使用`--privileged`标志运行容器。

例如，不要以如下方式启动一个容器：

```text
docker run --interactive --tty --privileged centos /bin/bash
```

**影响**

除了默认值之外，Linux内核功能将不能在容器内使用。

**默认值**

`False`

**参考文献**

```text
1. https://docs.docker.com/reference/commandline/cli
```

### 5.5 不要在容器上挂载敏感的主机系统目录（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

敏感的主机系统目录（如下面）不应该被允许作为容器卷挂载，特别是在读写模式下。

```text
/
/boot
/dev
/etc
/lib
/proc
/sys
/usr
```

**缘由**

如果敏感目录以读写模式挂载，则可以对敏感目录中的文件进行更改。 这些更改可能会降低安全隐患，或可能导致Docker主机处于无法保证的受损状态。

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: Volumes={{ .Mounts}}'
```

上述命令将返回当前映射目录的列表以及是否以读写模式为每个容器实例挂载。

**修正**

不要将主机敏感目录挂载到在容器上，特别是在读写模式下。

**影响**

无

**默认值**

Docker默认为读写卷，但也可以只读目录。 默认情况下，容器上不会安装敏感的主机目录。

**参考文献**

```text
1. https://docs.docker.com/userguide/dockervolumes
```

### 5.6 不要在容器中运行ssh（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

SSH服务不应该在容器内运行。 您应该SSH进入Docker主机，并使用nsenter工具从远程主机进入容器。

**缘由**

在容器中运行SSH会增加安全管理的复杂性：

* 难以管理SSH服务的访问策略和安全合规性
* 很难管理各个容器中的密钥和密码
* 难以管理SSH服务的安全升级

可以在没有使用SSH的情况下使用shell访问容器，应该避免不必要地增加安全管理的复杂性。

**审计**

步骤1：通过执行以下命令列出所有正在运行的容器实例：

```text
docker ps --quiet
```

步骤2：对于每个容器实例，执行以下命令：

```text
docker exec $INSTANCE_ID ps -el
```

确保没有SSH服务进程。

**修正**

从容器中卸载SSH服务，并使用nsenter或任何其他命令（如docker exec或docker attach）与容器实例进行交互。

```text
docker exec --interactive --tty $INSTANCE_ID sh
```

或者

```text
docker attach $INSTANCE_ID
```

**影响**

无

**默认值**

默认情况下，SSH服务不在容器中运行。 每个容器只允许一个进程。

**参考文献**

```text
1. http://blog.docker.com/2014/06/why-you-dont-need-to-run-sshd-in-docker/
```

### 5.7 不要给容器映射特权端口（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

低于1024的TCP / IP端口号被认为是特权端口。 普通用户和进程不允许出于各种安全原因使用它们。 Docker允许一个容器端口映射到一个特权端口。

**缘由**

默认情况下，如果用户没有明确声明容器端口为主机端口映射，则Docker将自动并正确地将容器端口映射到主机上的49153- 65535块中可用端口。 但是，如果用户显式声明，则允许将容器端口映射到主机上的特权端口。 这是因为容器是使用NET\_BIND\_SERVICE Linux内核功能执行的，并不限制特权端口映射。 特权端口接收和传输各种敏感和特权数据。 允许容器使用它们会带来严重的影响。

**审计**

通过执行以下命令列出所有正在运行的容器实例及其端口映射：

```text
docker ps --quiet | xargs docker inspect --format '{{ .Id }}: Ports={{.NetworkSettings.Ports }}'
```

查看列表并确保容器端口未映射到低于1024的主机端口号。

**修正**

启动容器时不要将容器端口映射到主机特权端口。 另外，确保在Dockerfile中没有托管特权端口映射的声明。

**影响**

无

**默认值**

默认情况下，允许将一个容器的端口映射到主机上的一个特权端口。

**注意：在某些情况下，您想要映射特权端口，因为如果禁止它，则相应的应用程序必须在容器之外运行。** **例如：HTTP和HTTPS负载均衡器必须分别绑定80 / tcp和443 / tcp。 禁止映射特权端口有效地禁止在容器中运行这些端口，并且要求使用外部负载平衡器。 在这种情况下，这些容器实例应该被标记为这个建议的例外。**

**参考文献**

```text
1. http://docs.docker.com/articles/networking/#binding-ports
2. https://www.adayinthelifeof.nl/2012/03/12/why-putting-ssh-on-another-port-
than-22-is-bad-idea
```

### 5.8 打开容器上只需要的端口（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

容器镜像的Dockerfile定义了容器实例上默认打开的端口。 端口列表可能与您在容器中运行的应用程序相关，也可能不相关。

**缘由**

一个容器可以运行Dockerfile中为其镜像定义的端口，或者可以任意传递运行时参数来打开一个端口列表。 此外，Dockerfile可能会经历各种更改，并且暴露的端口列表可能与您在容器中运行的应用程序相关，也可能不相关。 打开不需要的端口增加了容器、容器化应用程序的攻击面。 作为推荐的做法，不要打开不需要的端口。

**审计**

通过执行以下命令列出所有正在运行的容器实例及其端口映射：

```text
docker ps --quiet | xargs docker inspect --format '{{ .Id }}: Ports={{.NetworkSettings.Ports }}'
```

查看列表并确保映射的端口是容器真正需要的端口。

**修正**

修复容器镜像的Dockerfile，以便仅由您的容器化应用程序公开所需的端口。 你也可以通过在启动容器时不使用`'-P'`（UPPERCASE）或`'--publish-all'`标志来完全忽略在Dockerfile中定义的端口列表。 使用`'-p'`（小写）或`'--publish'`标志来明确定义特定容器实例所需的端口。 例如，

```text
docker run --interactive --tty --publish 5000 --publish 5001 --publish 5002 centos
/bin/bash
```

**影响**

无

**默认值**

默认情况下，在使用`“-P”`或`“--publish-all”`标志运行容器时，会打开在镜像Dockerfile中`EXPOSE`指令下的列出的所有端口。

**参考文献**

```text
1. https://docs.docker.com/articles/networking/#binding-ports
```

### 5.9 不要共享主机的网络名称空间（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

设置为`“--net = host”`时，容器上的联网模式将跳过将容器放置在单独的网络堆栈中。 实质上，这个选择告诉Docker不要容器化容器网络。 这将在网络方面意味着容器处于Docker主机的“外部”，并完全访问其网络接口。

**缘由**

这是潜在的危险。 它允许容器进程像任何其他 `root`进程一样打开低编号的端口。 它还允许容器访问Docker主机上的D-bus 等网络服务。 因此，容器进程可能会做出意想不到的事情，比如关闭Docker主机。 你不应该使用这个选项。

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: NetworkMode={{.HostConfig.NetworkMode }}'
```

如果上述命令返回`“NetworkMode = host”`，则意味着在启动容器时传递了`“--net = host”`选项。 这是不符合基线的。 它应该返回 `bridge`, `none`, 或 `container:$Container_Instance`。

**修正**

启动容器时不要传递`'--net = host'`选项。

**影响**

无

**默认值**

默认情况下，容器连接到Docker `bridge`。

**参考文献**

```text
1. http://docs.docker.com/articles/networking/#how-docker-networks-a-container 
2. https://github.com/docker/docker/issues/6401
```

### 5.10 限制容器的内存使用量（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

默认情况下，Docker主机上的所有容器均等共享资源。 通过使用Docker主机的资源管理功能，例如内存限制，您可以控制容器可能消耗的内存量。

**缘由**

默认情况下，容器可以使用主机上的所有内存。 您可以使用内存限制机制来防止由于一个容器消耗了所有主机资源而导致的拒绝服务，以致同一主机上的其他容器无法执行其预期的功能。 对内存没有限制可能导致一个容器容易使整个系统不稳定并因此不可用的问题。

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: Memory={{.HostConfig.Memory }}'
```

如果上述命令返回0，则意味着内存限制不到位。 如果上面的命令返回一个非零值，这意味着内存限制已经到位。

**修正**

根据需要只用尽可能多的内存来运行容器。 始终使用`'—memory'`参数运行容器。 你应该以如下方式启动容器：

```text
$> docker run <Run arguments> --memory <memory-size> <Container Image Name or ID> <Command>
```

例如，

```text
docker run --interactive --tty --memory 256m centos /bin/bash
```

在上面的示例中，容器启动的内存限制为256 MB。 **注意：请注意，如果内存限制到位，下面命令的输出将以科学记数法返回值。**

```text
docker inspect --format='{{.Config.Memory}}' 7c5a2d4c7fe0
```

上面的`7c5a2d4c7fe0`为容器ID。

例如，如果上述容器实例的内存限制设置为256 MB，则上述命令的输出将是2.68435456e + 08而不是256m。 您应该使用科学计算器或编程方法来转换此值。

**影响**

如果你没有设置适当的限制，容器过程可能不得不饿死。

**默认值**

默认情况下，Docker主机上的所有容器均等共享资源。 没有强制执行内存限制。

**参考文献**

```text
1. https://goldmann.pl/blog/2014/09/11/resource-management-in-docker/ 
2. http://docs.docker.com/reference/commandline/cli/#run
3. https://docs.docker.com/articles/runmetrics/
```

### 5.11 适当设置容器CPU的优先级（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

默认情况下，Docker主机上的所有容器均等共享资源。 通过使用Docker主机的资源管理功能，例如CPU共享，可以控制容器可能使用的主机CPU资源。

**缘由**

默认情况下，CPU时间在容器间平均分配。 如果需要，为了控制容器实例中的CPU时间，可以使用CPU共享功能。 CPU共享允许将一个容器优先于另一个容器，并且禁止较低优先级的容器更频繁地声明CPU资源。 这确保高优先级的容器更好地服务。

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: CpuShares={{.HostConfig.CpuShares }}'
```

如果上述命令返回0或1024，则意味着CPU份额不到位。 如果上述命令返回非1024、非零值，则意味着CPU份额已到位。

**修正**

管理您容器之间的CPU份额。 为此，请使用`'--cpu-shares'`参数启动容器。 您可以如下方式启动容器：

```text
$> docker run <Run arguments> --cpu-shares <CPU shares> <Container Image Name or ID> <Command>
```

例如，

```text
docker run --interactive --tty --cpu-shares 512 centos /bin/bash
```

在上面的例子中，容器以其他容器使用的50％的CPU份额启动。 所以，如果另一个容器具有80％的CPU份额，则该容器将具有40％的CPU份额。

**注意：默认情况下，每个新的容器将有1024个CPU份额。 但是，如果运行审计部分中提到的命令，则此值显示为“0”。**

或者：

1. 导航到`/sys/fs/cgroup/cpu/system.slice/`目录。
2. 使用`“docker ps”`命令检查您的容器实例ID。
3. 现在，在上面的目录中（在步骤1），你将有一个目录的名称为：

   `'docker- <Instance ID> .scope'`例如`'docker-4acae729e8659c6be696ee35b2237cc1fe4edd2672e9186434c5116e1a6fbed6.scope'`。 导航到这个目录。

   最新版没有此目录，直接下一步。

4. 您将找到一个名为`“cpu.shares”`的文件。 执行`'cat cpu.shares'`。 这将始终为您提供基于系统的CPU份额值。 因此，即使在`“docker run”`命令中没有使用`“-c”`或`“--cpu-shares”`参数配置的CPU份额，该文件的值也将为`“1024”`。

如果我们将一个容器的CPU份额设置为512，则与其他容器相比，它将获得一半的CPU时间。 所以，以1024为100％，然后做快速的数学推导出你应该为各自的CPU份额设置的数字。 例如，如果要设置50％，则使用512，如果要设置25%，则使用256。

**影响**

如果您没有设置正确的CPU份额，那么如果主机上的资源不可用，则容器进程可能不得不饿死。 如果主机上的CPU资源空闲，则CPU份额不会对容器可能使用的CPU进行任何限制。

**默认值**

默认情况下，Docker主机上的所有容器均等共享资源。 没有执行CPU份额。

**参考文献**

```text
1. https://goldmann.pl/blog/2014/09/11/resource-management-in-docker/ 
2. http://docs.docker.com/reference/commandline/cli/#run
3. https://docs.docker.com/articles/runmetrics/
```

### 5.12 将容器的根文件系统挂载为只读（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

容器的根文件系统应该被视为“黄金镜像”，并且应该避免对根文件系统的任何写入。 你应该明确地定义一个容器的卷来写入。

**缘由**

你不应该在容器中写数据。 应该明确定义和管理属于容器的数据卷。 这在管理员希望控制开发人员写文件和错误的情况下很有用。 另外，这还有其他的好处，例如：

* 这基础结构不会变化
* 由于无法写入容器实例，因此不需要审计实例差异
* 因为实例不能被篡改或写入，减少安全攻击媒介
* 能够使用基于卷的备份，而无需备份任何实例

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: ReadonlyRootfs={{.HostConfig.ReadonlyRootfs }}'
```

如果上述命令返回`“true”`，则意味着根文件系统是只读的。 如果上述命令返回`“false”`，则表示根文件系统是可写的。

此外，您可以使用下面的命令来查找容器实例与其相应镜像之间的差异。

```text
docker diff $INSTANCE_ID
```

**修正**

添加一个`“--read-only”`标志，允许将容器的根文件系统作为只读进行挂载。 这可以与卷组合使用，以强制容器的进程只写入将被持久化的位置。 您应该以如下方式运行容器：

```text
$> docker run <Run arguments> --read-only -v <writable-volume> <Container Image Name or ID> <Command>
```

例如，

```text
docker run --interactive --tty --read-only --volume /centdata centos /bin/bash
```

这将使用只读的根文件系统运行容器，并使用`“centdata”`作为容器卷进行写入。

**影响**

容器根文件系统将不可写入。 您应该明确地为容器定义一个卷来写入。

**默认值**

默认情况下，容器的根文件系统是可写的，允许进程在任何地方写文件。

**参考文献**

```text
1. http://docs.docker.com/reference/commandline/cli/#run
```

### 5.13 将传入的容器流量绑定到特定的主机接口（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

默认情况下，Docker容器可以连接到外部世界，但是外部世界不能连接到容器。 每个传出连接似乎都来自主机自己的一个IP地址。 只允许通过主机上的特定外部接口联系容器服务。

**缘由**

如果主机上有多个网络接口，则容器可以接受任何网络接口上暴露端口的连接。 这可能是不希望的，可能不安全。 很多时候，一个特定的接口暴露在外部，在这些接口上运行诸如入侵检测，入侵防御，防火墙，负载平衡等服务来屏蔽传入的公共流量。 因此，你不应该接受任何接口上的传入连接。 您应该只允许来自特定外部接口的传入连接。

**审计**

通过执行以下命令列出所有正在运行的容器实例及其端口映射：

```text
docker ps --quiet | xargs docker inspect --format '{{ .Id }}: Ports={{.NetworkSettings.Ports }}'
```

查看列表并确保公开的容器端口绑定到特定的接口，而不是通配符IP地址 - `“0.0.0.0”`。

例如， 如果上面的命令返回如下，那么容器可以接受指定端口49153上的任何主机接口上的连接，这是不符合基线的。

```text
Ports=map[443/tcp:<nil> 80/tcp:[map[HostPort:49153 HostIp:0.0.0.0]]]
```

但是，如果将暴露的端口连接到主机上的特定接口（如下所示），则此基线将根据需要进行配置并符合规定。

```text
Ports=map[443/tcp:<nil> 80/tcp:[map[HostIp:10.2.3.4 HostPort:49153]]]
```

**修正**

将容器端口绑定到所需主机端口上的特定主机接口。 例如，

```text
docker run --detach --publish 10.2.3.4:49153:80 nginx
```

在上面的示例中，容器端口80绑定到49153上的主机端口，并且只接受来自10.2.3.4外部接口的入站连接。

**影响**

无

**默认值**

默认情况下，Docker公开的容器端口为`0.0.0.0`，该通配符IP地址将与主机上任何可能的入网网络接口相匹配。

**参考文献**

```text
1. https://docs.docker.com/articles/networking/#binding-container-ports-to-the-host
```

### 5.14 将'on-failure'容器重启策略设置为5（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

在`“docker run”`命令中使用`“--restart”`标志，您可以指定一个重启策略，以便在退出时应该或不应该重启容器。 您应该选择`“on- failure”`重新启动策略，并将重新启动尝试次数限制为5次。

**缘由**

如果无限期地继续尝试启动容器，可能会导致主机上的拒绝服务。 这可能是一个简单的方法来做分布式拒绝服务攻击，特别是如果你在同一个主机上有很多容器。 此外，忽略容器的退出状态和`“always"`尝试重新启动容器会导致无法调查容器被终止的根本原因。 如果一个容器被终止，您应该调查它的原因，而不是试图无限期地重启它。 因此，建议使用`“on-failure”`重新启动策略，并将其限制为最多5次重新启动尝试。

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}:RestartPolicyName={{ .HostConfig.RestartPolicy.Name }} MaximumRetryCount={{.HostConfig.RestartPolicy.MaximumRetryCount }}'
```

* 如果上面的命令返回`'RestartPolicyName = always'`，那么系统没有按照需要进行配置，因此是不符合基线的。
* 如果以上命令返回`“RestartPolicyName = no”`或`“RestartPolicyName =”`，则重新启动策略将不会被使用，并且容器将永远不会被重新启动。 这个是不适用的，可以被认为是合规的。
* 如果上述命令返回`“RestartPolicyName = on-failure”`，则通过查看`“MaximumRetryCount”`来验证重新启动尝试的次数是否设置为5或更少。

**修正**

如果一个容器需要自己重新启动，那么以如下方式启动容器：

```text
$> docker run <Run arguments> --restart=on-failure:5 <Container Image Name or ID> <Command>
```

例如，

```text
docker run --detach --restart=on-failure:5 nginx
```

**影响**

容器将尝试重新启动5次。

**默认值**

默认情况下，容器未配置重新启动策略。 因此，容器不会尝试重新启动它们自己。

**参考文献**

```text
1. http://docs.docker.com/reference/commandline/cli/#restart-policies
```

### 5.15 不共享主机的进程名称空间（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

进程ID（PID）命名空间隔离进程ID号空间，这意味着不同PID名称空间中的进程可以具有相同的PID。 这是容器和主机之间的进程级隔离。

**缘由**

PID名称空间提供进程的分离。 PID命名空间删除了系统进程的视图，并允许包括PID 1在内的进程ID被重用。如果主机的PID名称空间与容器共享，基本上允许容器内的进程查看主机系统上的所有进程 。 这打破了主机和容器之间进程级别隔离的好处。 有人访问容器可以最终知道主机系统上运行的所有进程，甚至可以从容器内杀死主机系统进程。 这可能是灾难性的。 因此，不要与容器共享主机的进程名称空间。

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: PidMode={{.HostConfig.PidMode }}'
```

如果上述命令返回`“host”`，则表示主机PID名称空间与容器共享，否则符合规定。

**修正**

不要用`'--pid = host'`参数启动一个容器。

例如，不要以如下方式启动一个容器：

```text
docker run --interactive --tty --pid=host centos /bin/bash
```

**影响**

容器进程无法看到主机系统上的进程。 在某些情况下，您希望您的容器共享主机的进程名称空间。 例如，您可以使用`strace`或`gdb`等调试工具构建容器，但希望在调试容器内的进程时使用这些工具。 如果这是需要的，那么通过使用`“-p”`开关共享一个（或所需的）主机进程。

例如，

```text
docker run --pid=host rhel7 strace -p 1234
```

**默认值**

默认情况下，所有容器都启用PID名称空间，并且主机的进程名称空间不与容器共享。

**参考文献**

```text
1. https://docs.docker.com/reference/run/#pid-settings
2. http://man7.org/linux/man-pages/man7/pid_namespaces.7.html
```

### 5.16 不共享主机的IPC命名空间（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

IPC（POSIX / SysV IPC）命名空间提供了命名共享内存段，信号量和消息队列的分离。 因此，主机上的IPC命名空间不应与容器共享，并应保持独立。

**缘由**

IPC命名空间提供主机和容器之间的IPC分离。 如果主机的IPC名称空间与容器共享，基本上允许容器内的进程查看主机系统上的所有IPC。 这打破了主机和容器之间的IPC级隔离的好处。 有人访问容器可以最终操纵主机IPC。 这可能是灾难性的。 因此，不要与容器共享主机的IPC名称空间。

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: IpcMode={{.HostConfig.IpcMode }}'
```

如果上述命令返回`“host”`，则表示主机IPC命名空间与容器共享。 如果上述命令不返回任何内容，则主机的IPC名称空间不共享。 这个符合规定。

**修正**

不要用`'--ipc = host'`参数启动一个容器。 例如，不要以如下方式启动一个容器：

```text
docker run --interactive --tty --ipc=host centos /bin/bash
```

**影响**

共享内存段用于加速进程间通信。 它通常被高性能应用程序使用。 如果这样的应用程序被容器化为多个容器，则可能需要共享容器的IPC名称空间以实现高性能。 在这种情况下，您仍然应该共享容器特定的IPC名称空间，而不是主机IPC名称空间。 您可以与其他容器共享容器的IPC命名空间，如下所示：

```text
$> docker run <Run arguments> --ipc=container:$INSTANCE_ID <Container Image Name or ID> <Command>
```

例如，

```text
docker run --interactive --tty --ipc=container:e3a7a1a97c58 centos /bin/bash
```

**默认值**

默认情况下，所有容器都启用IPC命名空间，主机IPC命名空间不与任何容器共享。

**参考文献**

```text
1. https://docs.docker.com/reference/run/#ipc-settings
2. http://man7.org/linux/man-pages/man7/namespaces.7.html
```

### 5.17 不要将主机设备直接暴露给容器（Not Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

主机设备可以在运行时直接暴露给容器。 不要将主机设备直接暴露给容器，特别是对于不受信任的容器。

**缘由**

`--device`选项将主机设备暴露给容器，因此容器可以直接访问这些主机设备。 您不会要求容器以`privileged` 模式运行以访问和操作主机设备。 默认情况下，容器将能够`read`，`write`和`mknod`这些设备。 此外，容器可能会从主机中删除块设备。 因此，不要直接将主机设备暴露给容器。 如果想把主机设备暴露给一个容器，适当地使用共享权限：

* r - read only
* w - writable
* m - mknod allowed

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: Devices={{.HostConfig.Devices }}'
```

以上命令将列出每个设备以下信息：

* `CgroupPermissions` - 例如, rwm
* `PathInContainer` - 容器内的设备路径
* `PathOnHost` - 主机上的设备路径

验证是否需要从容器中访问主机设备，并且正确设置所需的权限。 如果上述命令返回`[]`，那么容器不能访问主机设备。 这个可以被认为是合规的。

**修正**

不要将主机设备直接暴露于容器。 如果需要将主机设备暴露给容器，请使用正确的一组权限： 例如，不要以如下方式启动一个容器：

```text
docker run --interactive --tty --device=/dev/tty0:/dev/tty0:rwm --
device=/dev/temp_sda:/dev/temp_sda:rwm centos bash
```

例如，以正确的权限共享主机设备：

```text
docker run --interactive --tty --device=/dev/tty0:/dev/tty0:rw --
device=/dev/temp_sda:/dev/temp_sda:r centos bash
```

**影响**

您将无法直接在容器内使用主机设备。

**默认值**

默认情况下，没有主机设备暴露于容器。 如果您不提供共享权限并选择将主机设备公开到容器，则主机设备将具有`read`, `write`和 `mknod`权限。

**参考文献**

```text
1. http://docs.docker.com/reference/commandline/cli/#run
```

### 5.18 只在需要的时候在运行时覆盖默认的ulimit（Not Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

默认的ulimit是在Docker daemon级别设置的。 但是，如果需要，您可以在容器运行时重写默认的ulimit设置。

**缘由**

`ulimit`提供了对shell可用的资源以及由它启动的进程的控制。 设置系统资源限制使您免于 fork炸弹等许多灾难。 有时候，即使友好的用户和合法的进程也会过度使用系统资源，从而导致系统无法使用。 应该遵守Docker daemon级别设置的默认ulimit。 如果默认的ulimit设置不适用于特定的容器实例，则可以将它们覆盖作为例外。 但是，不要这样做。 如果大部分容器实例都覆盖默认的ulimit设置，请考虑将默认的ulimit设置更改为适合您需要的设置。

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: Ulimits={{.HostConfig.Ulimits }}'
```

上述命令应该为每个容器实例返回`Ulimits = <no value>`，除非有例外，并且需要覆盖默认的ulimit设置。

**修正**

如果需要，才覆盖默认的ulimit设置。 例如，要覆盖默认的ulimit设置，请按以下方式启动一个容器：

```text
docker run --ulimit nofile=1024:1024 --interactive --tty centos /bin/bash
```

**影响**

如果ulimits设置不正确，可能无法实现所需的资源控制，甚至可能导致系统无法使用。

**默认值**

容器实例继承在Docker daemon级别设置的默认ulimit设置。

**参考文献**

```text
1. http://docs.docker.com/reference/commandline/cli/#setting-ulimits-in-a- container
2. Command: man setrlimit
3. http://www.oreilly.com/webops-perf/free/files/docker-security.pdf
```

**注意：** **ulimit中有多个可供选择的选项。 你可以控制的几个重要的限制是CPU（以秒为单位的CPU时间限制），fsize（文件进程可以创建的最大大小），memlock（可能锁定到RAM的最大内存字节数），nproc（mx进程数 ）等，完整列表可以见命令“man setrlimit”**

### 5.19 不要将挂载的传播模式设为共享\(Scored\)

**配置适用性**

* **Level 1- Docker**

**描述**

挂载的传播允许在容器上以共享，从属或私有的模式来挂载卷，如果没有必要的话请不要使用共享模式来挂载卷。

**缘由**

共享挂载将在所有挂载中进行复制，并且将在任何挂载点进行的更改传播到所有安装。 以共享模式挂载卷不会限制其他容器的挂载并对该卷进行更改。 如果挂载的卷对变化敏感，这可能是灾难性的。 所以如果不是真的有必要的话不要将挂载的传播模式设为共享。

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}:Propagation={{range $mnt := .Mounts}} {{json $mnt.Propagation}} {{end}}'
```

上述命令将返回已挂载卷的传播模式。 除非需要，否则传播模式不应设置为“共享”。 如果没有挂载，上面的命令可能会引发错误。 在这种情况下，这个是不适用的。

**修正**

不要以共享的传播模式挂载卷，例如，不要以如下方式启动容器：

```text
docker run <Run arguments> --volume=/hostPath:/containerPath:shared <Container Image Name or ID> <Command>
```

**影响**

无

**默认值**

默认情况下，容器的挂载都是私有的。

**参考文献**

```text
1. https://github.com/docker/docker/pull/17034
2. https://docs.docker.com/engine/reference/run/
3. https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt
```

### 5.20 不要共享主机的UTS命名空间（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

UTS命名空间提供两个系统标识符的隔离：主机名和NIS域名。 它用于设置在该名称空间中正在运行的进程可见的主机名和域。 在容器中运行的进程通常不需要知道主机名和域名。 因此，命名空间不应与主机共享。

**缘由**

与主机共享UTS命名空间会对容器提供完全权限以更改主机的主机名。 这是不安全的，不应该被允许。

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: UTSMode={{.HostConfig.UTSMode }}'
```

如果上述命令返回`“host”`，则意味着主机UTS名称空间与容器共享，并且不符合基线。 如果上述命令不返回任何内容，则主机的UTS名称空间不会共享。 是符合这个基线的。

**修正**

不要使用`“--uts = host”`参数启动容器。 例如，不要以如下方式启动一个容器：

```text
docker run --rm --interactive --tty --uts=host rhel7.2
```

**影响**

无

**默认值**

默认情况下，所有容器都启用了UTS命名空间，并且主机UTS命名空间不与任何容器共享。

**参考文献**

```text
1. https://docs.docker.com/engine/reference/run/
2. http://man7.org/linux/man-pages/man7/namespaces.7.html
```

### 5.21 不要禁用默认seccomp配置文件（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

Seccomp过滤为进程指定传入系统调用的过滤器提供了一种手段。 默认的Docker seccomp配置文件基于白名单，并允许311个系统调用阻止所有其他的配置。 它不应该被禁用，除非它阻碍你的容器应用程序的使用。

**缘由**

大量的系统调用暴露于每个用户级进程，其中许多系统调用在整个生命周期中都未被使用。 大多数应用程序不需要所有的系统调用，因此可以通过减少可用的系统调用来获益。 减少的一组系统调用减少了暴露给应用程序的整个内核表面，从而提高应用程序的安全性。

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: SecurityOpt={{.HostConfig.SecurityOpt }}'
```

上面的命令应该返回`“<no value>”`或您修改的seccomp配置。 如果它返回`[seccomp：unconfined]`，这意味着不符合此基线，容器没有使用任何seccomp配置文件。

**修正**

默认情况下，seccomp配置文件已启用。 除非要修改和使用修改后的seccomp配置文件，否则不需要执行任何操作。

**影响**

使用Docker 1.10或更高版本，默认的seccomp配置文件阻塞系统调用，而不管传递给容器的`--cap-add`。 在这种情况下，您应该创建自己的自定义seccomp配置文件。 您也可以通过在`docker run`上传递`--security- opt = seccomp：unconfined`来禁用默认的seccomp配置文件。

**默认值**

运行容器时，除非使用`--security-opt`选项覆盖，否则它将使用默认配置文件。

**参考文献**

```text
1. http://blog.scalock.com/new-docker-security-features-and-what-they-mean- seccomp-profiles
2. https://docs.docker.com/engine/reference/run/
3. https://github.com/docker/docker/blob/master/profiles/seccomp/default.json
4. https://docs.docker.com/engine/security/seccomp/
5. https://www.kernel.org/doc/Documentation/prctl/seccomp_filter.txt
6. https://github.com/docker/docker/issues/22870
```

### 5.22 不要执行具有特权选项的docker exec命令（Scored）

**配置适用性**

* **Level 2- Docker**

**描述**

不要使用`--privileged`选项执行`docker exec`。

**缘由**

在`docker exec`中使用`--privileged`选项可为命令提供扩展的Linux功能。 这可能是不安全的，特别是在运行具有丢弃功能或增强限制的容器时。

**审计**

如果按照第1节的规定启用了审计，则可以使用以下命令过滤掉使用`--privileged`选项的`docker exec`命令。

```text
ausearch -k docker | grep exec | grep privileged
```

**修正**

在`docker exec`命令中不要使用`--privileged`选项。

**影响**

无。 如果您需要在容器内增强功能，请使用所需功能运行容器。

**默认值**

默认情况下，docker exec命令运行时没有`--privileged` 选项。

**参考文献**

```text
1. https://docs.docker.com/engine/reference/commandline/exec/
```

### 5.23 不要使用user选项的执行docker exec命令（Scored）

**配置适用性**

* **Level 2- Docker**

**描述**

docker exec不要使用`--user`选项。

**缘由**

在`docker exec`中使用`--user`选项以该用户身份执行容器内的命令。 这可能是不安全的，特别是在运行具有丢弃功能或增强限制的容器时。 例如，假设你的容器是以tomcat用户（或任何其他非root用户）的身份运行的，那么可以通过`docker exec` `--user = root`选项以`root`身份运行一个命令。 这可能是危险的。

**审计**

如果您按照第1节的规定启用了审计，则可以使用以下命令过滤掉使用了`--user`选项的`docker exec`命令。

```text
ausearch -k docker | grep exec | grep user
```

**修正**

不要使用`--user`选项执行`docker exec`

**影响**

无

**默认值**

默认情况下，执行`docker exec`时，没有使用`--user`选项。

**参考文献**

```text
1. https://docs.docker.com/engine/reference/commandline/exec/
```

### 5.24 确认cgroup的用法\(Scored\)

**配置适用性**

* **Level 1- Docker**

**描述**

可以在容器运行时附加到特定的cgroup，确认cgroup使用将确保容器在定义的cgroup下运行。

**缘由**

系统管理员通常定义容器应该在哪个cgroup下运行。 即使cgroups没有被系统管理员明确定义，默认情况下，容器会在docker cgroup下运行。 在运行时，可以附加到不同于预期使用的cgroup之外的其他cgroup。 这个用法应该被监视和确认。 通过附加到一个不同的cgroup，可能会被授予额外的权限和资源，因此可能被证明是不安全的。

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: CgroupParent={{.HostConfig.CgroupParent }}'
```

上述命令将返回容器正在运行的cgroup。 如果它是空的，这意味着容器在默认的`docker cgroup`下运行。 在这种情况下，是符合基线的。 如果发现容器正在运行非预期的cgroup，是不符合此基线的。

**修正**

除非需要，否则不要在docker run命令中使用`--cgroup-parent`选项。

**影响**

无

**默认值**

默认情况下，容器在`docker` cgroup下运行。

**参考文献**

```text
1. https://docs.docker.com/engine/reference/run/#specifying-custom-cgroups 
2.https://access.redhat.com/documentation/en_US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/ch01.html
```

### 5.25 限制容器获得额外的权限（得分）

**配置适用性**

* **Level 1- Docker**

**描述**

限制容器通过`suid`或`sgid`位获得额外的权限。

**缘由**

一个进程可以在内核中设置`no_new_priv`位。其值贯穿于 `fork`，\`\`clone`和`execve\`。\` no\_new\_priv\`位确保进程或其子进程不会通过 \`suid\`或\`sgid\`位获得任何额外的权限。 这样，很多危险的操作变得不那么危险，因为不可能破坏特权二进制文件。

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: SecurityOpt={{.HostConfig.SecurityOpt }}'
```

上述命令应该返回当前为容器配置的所有安全选项。 `no-new-privileges` 也应该是其中之一。

**修正**

以如下方式启动容器：

```text
docker run <run-options> --security-opt=no-new-privileges <IMAGE> <CMD>
```

例如，

```text
docker run --rm -it --security-opt=no-new-privileges ubuntu bash
```

**影响**

`no_new_priv`防止像SELinux这样的LSM转换到不允许访问当前进程的进程标签。

**默认值**

默认情况下，新的权限不受限制。

**参考文献**

```text
1. https://github.com/projectatomic/atomic-site/issues/269
2. https://github.com/docker/docker/pull/20727
3. https://www.kernel.org/doc/Documentation/prctl/no_new_privs.txt 
4. https://lwn.net/Articles/475678/
5. https://lwn.net/Articles/475362/
```

### 5.26 在容器运行时对容器做健康检查\(Scored\)

**配置适用性**

* **Level 1- Docker**

**描述**

如果容器镜像没有定义`HEALTHCHECK`指令，则在容器运行时使用`--health-cmd`参数来检查容器运行状况。

**缘由**

安全的三要素之一是可用性。如果您正在使用的容器镜像没有预定义的`HEALTHCHECK`指令，请使用`--health-cmd`参数在运行时检查容器运行状况。 根据报告的健康状况，您可以采取必要的措施。

**审计**

运行以下命令并确保所有容器都报告运行状况：

```text
docker ps --quiet | xargs docker inspect --format '{{ .Id }}: Health={{.State.Health.Status }}'
```

**修正**

使用`--health-cmd`和其他参数运行容器。 例如，

```text
docker run -d --health-cmd='stat /etc/passwd || exit 1' nginx
```

**影响**

无

**默认值**

默认情况下，容器运行时并没有做健康检查。

**参考文献**

```text
1. https://github.com/docker/docker/pull/22719
```

### 5.27 确保docker命令始终获取最新版本的镜像（Not Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

始终确保在存储库中使用最新版本的映像，而不是缓存的旧版本。

**缘由**

多个docker命令，比如`docker pull`，`docker run`等，存在一个已知问题，即默认情况下它们会提取镜像的本地副本（如果存在的话），即使在上游存储库中存在具有“相同标记”的更新版本的镜像 “。 这可能会导致使用较老的和易受攻击的镜像。

**审计**

步骤1：打开您的镜像库，并列出您正在检查的镜像的镜像版本历史记录。

步骤2：观察`docker pull`命令启动时的状态，如果状态显示为`Image is up to date,`，则表示您正在获取的镜像是缓存版本。

步骤3：将正在运行的镜像版本与存储库中报告的最新版本匹配，以确定您是在运行缓存版本还是最新版本。

**修正**

使用适当的版本固定机制（默认分配的最新标记仍然容易受到缓存攻击），以避免提取缓存的旧版本。 版本固定机制也应该用于基本映像，软件包和整个映像。 您可以根据您的要求自定义版本固定规则。

**影响**

无

**默认值**

默认情况下，除非使用了版本固定机制或清除了本地缓存，否则docker命令会提取本地副本。

**参考文献**

```text
1. https://github.com/docker/docker/pull/16609
```

### 5.28 使用PID cgroup限制（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

在容器运行时使用 `--pids-limit` 标志

**缘由**

攻击者可以在容器内用单个命令形成fork炸弹。 这种fork炸弹可能会使整个系统崩溃，并需要重新启动主机才能使系统再次正常运行。 PID cgroup `--pids-limit`将通过限制在给定时间内容器内可能发生的fork数量来防止这种攻击。

**审计**

运行以下命令并确保`PidsLimit`未设置为`0`或`-1`。 `0`或`-1`的`PidsLimit`表示可以同时在容器内fork任意数量的进程。

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: PidsLimit={{.HostConfig.PidsLimit }}'
```

**修正**

使用`--pids-limit`标志，以适当值启动容器。 例如，

```text
 docker run -it --pids-limit 100 <Image_ID>
```

在上面的例子中，在任何给定的时间允许运行的进程数被设置为100.在达到100个并发运行进程的限制之后，docker将限制任何新的进程创建。

**影响**

请根据需要设置PID值。 否则可能会使容器无法使用。

**默认值**

`--pids-limit`的默认值是0，这意味着fork的数量没有限制。 另外请注意，PID cgroup限制仅适用于内核版本4.3及以上的版本

**参考文献**

```text
1. https://github.com/docker/docker/pull/18697
2. https://docs.docker.com/engine/reference/commandline/run/
```

### 5.29 不要使用Docker的默认网桥docker0（Not Scored）

**配置适用性**

* **Level 2- Docker**

**描述**

不要使用Docker的默认网桥`docker0`。 使用用户定义的docker网络进行容器联网

**缘由**

Docker将以桥模式创建的虚拟接口连接到一个名为`docker0`的公共桥。 由于没有使用过滤，这种默认的网络模型容易受到ARP欺骗和MAC洪泛攻击。

**审计**

运行以下命令，并验证容器是否在用户定义的网络上，而不是默认的`docker0`网桥。

```text
docker network ls --quiet | xargs xargs docker network inspect --format '{{ .Name }}:{{ .Options }}'
```

**修正**

遵循Docker文档并设置用户定义的网络。 在定义的网络中运行所有容器。

**影响**

您必须管理用户自定义的网络。

**默认值**

默认情况下，docker在其`docker0`桥上运行容器。

**参考文献**

```text
1. https://github.com/nyantec/narwhal
2. https://arxiv.org/pdf/1501.02967
3. https://docs.docker.com/engine/userguide/networking/dockernetworks/
```

### 5.30 不要共享主机的用户名空间（Socred）

**配置适用性**

* **Level 1- Docker**

**描述**

不要与容器共享主机的用户名称空间。

**缘由**

用户名空间确保容器内的root进程将映射到容器外的非root进程。 因此，使用容器共享主机的用户命名空间不会将容器上的用户与主机上的用户隔离开来。

**审计**

运行下面的命令不会返回`UsernsMode`的任何值。 如果它返回一个`host`的值，这意味着主机用户名称空间与容器共享。

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: UsernsMode={{ .HostConfig.UsernsMode }}' 
```

**修正**

不要在主机和容器之间共享用户名空间。

**影响**

无

**默认值**

默认情况下，主机用户名称空间与容器没有启用共享。

**参考文献**

```text
1. https://docs.docker.com/engine/reference/commandline/run/#/run
2. https://events.linuxfoundation.org/sites/events/files/slides/User%20Namespaces%20-%20ContainerCon%202015%20-%2016-9-final_0.pdf 
3. https://github.com/docker/docker/pull/12648
```

### 5.31 不要将Docker套接字挂载到任何容器中（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

docker套接字（`docker.sock`）不应该挂载到容器内。

**缘由**

如果docker套接字被挂载到一个容器内，它将允许在容器中运行的进程执行可以完全控制docker主机的命令。

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: Volumes={{ .Mounts}}' | grep docker.sock
```

以上命令将返回`docker.sock`已经映射到容器之内的所有docker实例。

**修正**

确保没有容器将docker.sock挂载为卷。

**影响**

无

**默认值**

默认情况下，`docker.sock`不会挂载到容器中。

**参考文献**

```text
1. https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/
2. https://forums.docker.com/t/docker-in-docker-vs-mounting-var-run-docker-sock/9450/2
3. https://github.com/docker/docker/issues/21109
```

