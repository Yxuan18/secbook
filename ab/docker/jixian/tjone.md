# 推荐一

## 主机配置

本节将介绍为用于执行容器化工作负载主机应遵循的安全建议。

保护Docker主机并遵循基础设施安全最佳实践将为执行容器化工作负载的主机构建坚实可靠的基础。

### 1.1 为容器创建一个单独的分区（scored）

* **Level 1- Docker**

**描述**

所有Docker容器及其数据和元数据都存储在`/ var / lib / docker` 目录下。 默认情况下，`/ var / lib / docker` 将根据可用性挂载在 `/` 或 `/ var` 分区下。

**缘由**

Docker 将 `/ var / lib / docker` 作为存储所有Docker相关文件（包括 images）的默认目录。 这个目录可能会快速填满，Docker和主机将变的无法使用。 所以，建议创建一个单独的分区（逻辑卷）来存储Docker文件。

**审计**

在Docker主机上执行以下命令：

```text
grep /var/lib/docker /etc/fstab
```

这应该返回 `/ var / lib / docker` 挂载点分区的详细信息。

**修正**

对于新安装的系统，为 `/ var / lib / docker` 挂载点创建一个单独的分区。 对于已安装的系统，请使用逻辑卷管理器（LVM）创建分区。

**影响**

无

**默认值**

默认情况下，`/ var / lib / docker` 将根据可用性安装在 `/` 或 `/ var` 分区下。

**参考文献**

```text
1. http://www.projectatomic.io/docs/docker-storage-recommendation
```

### 1.2 加强容器主机（Not Scored）

**配置适用性**

* **Level 1- Linux Host OS**

**描述**

容器在Linux主机上运行。 容器主机可以运行一个或多个容器。 加强主机安全以减轻主机安全配置错误是非常重要的。

**缘由**

应该遵循基础设施安全最佳实践并加强您的主机操作系统。 强化主机系统将使得主机漏洞得到缓解。 否则，可能会导致安全风险和违规行为。

**审计**

确保遵循主机特定的安全准则。 向系统管理员询问当前主机系统符合哪个安全基线。 确保主机系统实际上符合主机特定的安全基线。

**修正**

可以考虑您的容器主机的各种CIS安全基线。 如果您有其他安全指导方针或法规要求要遵守，请在您的环境中按照适当的方式进行操作。 另外，你可以使用 `grsecurity` 和 `PaX` 运行一个内核。 这会在编译时和运行时增加许多安全检查。 它也被设计用于防御许多漏洞，具有强大的安全功能。 这些功能不需要特定于Docker的配置，因为这些安全功能适用于系统范围，与容器无关。

**影响**

无

**默认值**

默认情况下，主机具有出厂设置。 它没有强化。

**参考文献**

```text
1. https://docs.docker.com/articles/security/
2. https://benchmarks.cisecurity.org/downloads/multiform/index.cfm
3. http://docs.docker.com/articles/security/#other-kernel-security-features
4. https://grsecurity.net/
5. https://en.wikibooks.org/wiki/Grsecurity
6. https://pax.grsecurity.net/
7. http://en.wikipedia.org/wiki/PaX	
```

### 1.3 保持Docker 及时更新（Not Scored）

**配置适用性**

* **Level 1- Linux Host OS**

**描述**

Docker软件频繁发布，可解决安全漏洞，产品缺陷和引入新功能。 在这些产品更新中保留了一个选项卡，并在新的安全漏洞被修复或被认为是正确的时候会频繁升级。

**缘由**

通过及时进行Docker更新，Docker软件中的漏洞可以得到缓解。 受过教育的攻击者可能会在尝试访问或提升权限时利用已知的漏洞。 不安装常规的Docker更新可能会让您运行易受攻击的Docker软件。 这可能会导致权限提升，未经授权的访问或其他安全漏洞。 跟踪新版本并根据需要进行更新。

**审计**

执行以下命令并验证Docker版本是否为最新版本。 尽管这不是最新的版本。

```text
docker version
```

**修正**

跟踪Docker版本并根据需要进行更新。

**影响**

无

**默认值**

不适用

**参考文献**

```text
1. https://docs.docker.com/installation/
2. https://github.com/docker/docker/releases/latest
```

### 1.4 只允许可信用户控制Docker守护进程（Scored）

**配置适用性**

* **Level 1- Linux Host OS**

**描述**

Docker守护进程当前需要`“root”`权限。 添加到 `“docker”` 组的用户拥有充分的 `“root”`访问权限。

**缘由**

Docker允许在Docker主机和访客容器之间共享一个目录，而不会限制容器的访问权限。 这意味着您可以启动一个容器并将主机上的 `“/”` 目录映射到容器。 容器将能够不受任何限制地更改您的主机文件系统。 简而言之，这意味着只要是 `“docker”` 组的成员，然后在主机上启动一个映射 `/` 目录的容器，就可以获得更高的权限。

**审计**

在docker主机上执行以下命令，并确保只有受信任的用户是`“docker”` 组的成员。

```text
getent group docker
```

**修正**

从 `“docker”` 组中删除任何不受信任的用户。 此外，不要创建主机上的敏感目录到容器卷的映射。

**影响**

以普通用户身份建立和执行容器的权限将受到限制。

**默认值**

不适用

**参考文献**

```text
1. https://docs.docker.com/articles/security/#docker-daemon-attack-surface
2. https://www.andreas-jung.com/contents/on-docker-security-docker-group-considered-harmful
3. http://www.projectatomic.io/blog/2015/08/why-we-dont-let-non-root-users-run-docker-in-centos-fedora-or-rhel/

```

### 1.5 审计docker daemon（Scored）

**配置适用性**

* **Level 1- Linux Host OS**

**描述**

审计所有的Docker daemon活动。

**缘由**

除了审计常规的Linux文件系统和系统调用外，还要审计Docker daemon。 Docker daemon以 `“root”` 权限运行。 因此有必要审计其活动和使用情况。

**审计**

验证是否有Docker daemon 的审计规则。 例如，执行以下命令：

```text
auditctl -l | grep /usr/bin/docker
```

这应该列出Docker守护进程的规则。

**修正**

为Docker daemon 添加一条规则。 例如，在 `/etc/audit/audit.rules` 文件中添加如下行：

```text
-w /usr/bin/docker -k docker
```

然后，重新启动审计 daemon。 例如，

```text
service auditd restart
```

**影响**

审计会生成相当大的日志文件。 确保定期转移和归档。 此外，创建一个单独的审计分区，以避免填充根文件系统。

**默认值**

默认情况下，Docker daemon不被审计。

**参考文献**

```text
1. https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/chap-system_auditing.html
```

### 1.6 审计Docker文件和目录 - / var / lib / docker（Scored）

**配置适用性**

* **Level 1- Linux Host OS**

**描述**

审计 `/var/lib/docker`

**缘由**

除了审计您的常规Linux文件系统和系统调用外，还要审计所有Docker相关的文件和目录。 Docker daemon以 `“root”` 权限运行。 它的行为取决于一些关键的文件和目录。 `/ var / lib / docker` 就是一个这样的目录， 它包含有关容器的所有信息。 它必须被审计。

**审计**

验证是否存在与`/ var / lib / docker` 目录相对应的审计规则。

例如，执行以下命令：

```text
auditctl -l | grep /var/lib/docker
```

这应该列出 `/ var / lib / docker` 目录的规则。

**修正**

为 `/ var / lib / docker` 目录添加一条规则。

例如， 在/etc/audit/audit.rules文件中添加如下行：

```text
-w /var/lib/docker -k docker
```

然后，重新启动审计 daemon。 例如，

```text
service auditd restart
```

**影响**

审计会生成相当大的日志文件。 确保定期转移和归档。 此外，创建一个单独的审计分区，以避免填充根文件系统。

**默认值**

默认情况下，不会审计Docker相关的文件和目录。

**参考文献**

```text
 1.  https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/chap-system_auditing.html
```

### 1.7 审计Docker文件和目录 - / etc /docker（Scored）

**配置适用性**

* **Level 1- Linux Host OS**

**描述**

审计 `/etc/docker`。

**缘由**

除了审计您的常规Linux文件系统和系统调用外，还要审计所有Docker相关的文件和目录。 Docker deaemon 以 `“root”` 权限运行。 它的行为取决于一些关键的文件和目录。 `/ etc / docker` 就是一个这样的目录。 它拥有用于Docker daemon和Docker客户端之间TLS通信的各种证书和密钥。 它必须被审计。

**审计**

验证是否存在与 `/ etc / docker` 目录相对应的审计规则。

例如，执行以下命令：

```text
auditctl -l | grep /etc/docker
```

这应该列出 `/ etc / docker` 目录的规则。

**修正**

为 `/ etc / docker` 目录添加一条规则。

例如， 在 `/etc/audit/audit.rules`文件中添加如下行：

```text
-w /etc/docker -k docker
```

然后，重新启动审计 daemon。 例如，

```text
service auditd restart
```

**影响**

审计会生成相当大的日志文件。 确保定期转移和归档。 此外，创建一个单独的审计分区，以避免填充根文件系统。

**默认值**

默认情况下，不会审计Docker相关的文件和目录。

**参考文献**

```text
 1.  https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/chap-system_auditing.html
```

### 1.8 审计Docker文件和目录 - docker.service（Scored）

**配置适用性**

* **Level 1- Linux Host OS**

**描述**

如果可以，审计 `docker.service`

**缘由**

除了审计您的常规Linux文件系统和系统调用外，还要审计所有Docker相关的文件和目录。 Docker daemon 以 `“root”` 权限运行。 它的行为取决于一些关键的文件和目录。 `docker.service` 就是这样一个文件。 如果daemon 参数已被管理员更改，则 `docker.service`文件可能存在。 它拥有Docker daemon的各种参数。 如果可用，它必须被审计。

**审计**

步骤1：找出文件位置：

```text
systemctl show -p FragmentPath docker.service
```

步骤2：如果该文件不存在，则该基线不适用。 如果文件存在，请验证是否存在与该文件相对应的审计规则：

例如，执行以下命令：

```text
auditctl -l | grep docker.service
```

这应该根据其位置列出 `docker.service` 的规则。

**修正**

如果该文件存在，请为其添加规则。

例如， 在 `/etc/audit/audit.rules` 文件中添加如下行：\(注意：下面的路径根据实际改变\)

```text
-w /usr/lib/systemd/system/docker.service -k docker
```

然后，重新启动审计 daemon。 例如，

```text
service auditd restart
```

**影响**

审计会生成相当大的日志文件。 确保定期转移和归档。 此外，创建一个单独的审计分区，以避免填充根文件系统。

**默认值**

默认情况下，不会审计Docker相关的文件和目录。 `docker.service` 文件可能在系统上不可用。

**参考文献**

```text
 1.  https://access.redhat.com/documentation/en- US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/chap-system_auditing.html
```

### 1.9 审计Docker文件和目录 - docker.socket（Scored）

**配置适用性**

* **Level 1- Linux Host OS**

**描述**

如果可以，审计 `docker.socket`。

**缘由**

除了审计您的常规Linux文件系统和系统调用外，还要审计所有Docker相关的文件和目录。 Docker daemon以 `“root”` 权限运行。 它的行为取决于一些关键的文件和目录。 `docker.socket` 就是这样一个文件。 它拥有Docker daemon socket的各种参数。 如果可以，它必须被审计。

**审计**

步骤1：找出文件位置：

```text
systemctl show -p FragmentPath docker.socket
```

步骤2：如果该文件不存在，则该基线不适用。 如果文件存在，请验证是否存在与该文件相对应的审计规则：

例如，执行以下命令：

```text
auditctl -l | grep docker.socket
```

这应该根据其位置列出 `docker.socket` 的规则。

**修正**

如果该文件存在，请为其添加规则。

例如， 在 `/etc/audit/audit.rules`文件中添加如下行：\(注意：下面的路径根据实际改变\)

```text
-w /usr/lib/systemd/system/docker.socket -k docker
```

然后，重新启动审计 daemon。 例如，

```text
service auditd restart
```

**影响**

审计会生成相当大的日志文件。 确保定期转移和归档。 此外，创建一个单独的审计分区，以避免填充根文件系统。

**默认值**

默认情况下，不会审计Docker相关的文件和目录。 `docker.socket` 文件可能在系统上不可用。

**参考文献**

```text
 1.  https://access.redhat.com/documentation/en- US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/chap-system_auditing.html
```

### 1.10 审计Docker文件和目录 - /etc/default/docker（Scored）

**配置适用性**

* **Level 1- Linux Host OS**

**描述**

如果可以，审计 `/etc/default/docker`。

**缘由**

除了审计您的常规Linux文件系统和系统调用外，还要审计所有Docker相关的文件和目录。 Docker daemon 以`“root”` 权限运行。 它的行为取决于一些关键的文件和目录。 `/ etc / default / docker`就是这样一个文件。 它拥有Docker daemon的各种参数。 如果可以，它必须被审计。

**审计**

验证是否存在与 `/ etc / default / docker`文件相对应的审计规则。

例如，执行以下命令：

```text
auditctl -l | grep /etc/default/docker
```

这应该列出 `/ etc / default / docker` 文件的规则。

**修正**

为`/ etc / default / docker`文件添加一条规则。

例如， 在`/etc/audit/audit.rules` 文件中添加如下行：

```text
-w /etc/default/docker -k docker
```

然后，重新启动审计daemon。 例如，

```text
service auditd restart
```

**影响**

审计会生成相当大的日志文件。 确保定期转移和归档。 此外，创建一个单独的审计分区，以避免填充根文件系统。

**默认值**

默认情况下，不会审计Docker相关的文件和目录。 文件`/ etc / default / docker` 可能在系统上不可用。 在这种情况下，这个推荐基线是不适用的。

**参考文献**

```text
 1.  https://access.redhat.com/documentation/en- US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/chap-system_auditing.html

```

### 1.11 审计Docker文件和目录 - /etc/docker/daemon.json（Scored）

**配置适用性**

* Level 1- Linux Host OS

**描述**

如果可以，审计 `/etc/docker/daemon.json`。

**缘由**

除了审计您的常规Linux文件系统和系统调用外，还要审计所有Docker相关的文件和目录。 Docker daemon以 `“root”`权限运行。 它的行为取决于一些关键的文件和目录。`/etc/docker/daemon.json` 就是这样一个文件。 它拥有Docker daemon的各种参数。 如果可以，它必须被审计。

**审计**

验证是否存在与 `/etc/docker/daemon.json` 文件相对应的审计规则。

例如，执行以下命令：

```text
auditctl -l | grep /etc/docker/daemon.json
```

这应该列出 `/etc/docker/daemon.json` 文件的规则。

**修正**

为 `/etc/docker/daemon.json` 文件添加一条规则。

例如， 在 `/etc/audit/audit.rules`文件中添加如下行：

```text
-w /etc/docker/daemon.json -k docker
```

然后，重新启动审计daemon。 例如，

```text
service auditd restart
```

**影响**

审计会生成相当大的日志文件。 确保定期转移和归档。 此外，创建一个单独的审计分区，以避免填充根文件系统。

**默认值**

默认情况下，不会审计Docker相关的文件和目录。 文件 `/etc/docker/daemon.json` 可能在系统上不可用。 在这种情况下，这个推荐基线是不适用的。

**参考文献**

```text
1. https://access.redhat.com/documentation/en- US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/chap-system_auditing.html
2. https://docs.docker.com/engine/reference/commandline/daemon/#daemon- configuration-file
```

### 1.12 审计Docker文件和目录 - /usr/bin/docker-containerd（Scored）

**配置适用性**

* **Level 1- Linux Host OS**

**描述**

如果可以，审计 `/usr/bin/docker-containerd`。

**缘由**

除了审计您的常规Linux文件系统和系统调用外，还要审计所有Docker相关的文件和目录。 Docker daemon以`“root”` 权限运行。 它的行为取决于一些关键的文件和目录。`/usr/bin/docker-containerd` 就是这样一个文件。 Docker现在依靠 `containerd` 和`runC`来产生容器。 如果可以，它必须被审计。

**审计**

验证是否存在与`/usr/bin/docker-containerd`文件相对应的审计规则。

例如，执行以下命令：

```text
auditctl -l | grep /usr/bin/docker-containerd
```

这应该列出 `/usr/bin/docker-containerd` 文件的规则。

**修正**

为 `/usr/bin/docker-containerd` 文件添加一条规则。

例如， 在`/etc/audit/audit.rules`文件中添加如下行：

```text
-w /usr/bin/docker-containerd -k docker
```

然后，重新启动审计daemon。 例如，

```text
service auditd restart
```

**影响**

审计会生成相当大的日志文件。 确保定期转移和归档。 此外，创建一个单独的审计分区，以避免填充根文件系统。

**默认值**

默认情况下，不会审计Docker相关的文件和目录。 文件 `/usr/bin/docker- containerd` 可能在系统上不可用。 在这种情况下，这个推荐基线是不适用的。

**参考文献**

```text
1. https://access.redhat.com/documentation/en- US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/chap-system_auditing.html
2. https://github.com/docker/docker/pull/20662 
3. https://containerd.tools/

```

### 1.13 审计Docker文件和目录 - /usr/bin/docker-runc（Scored）

**配置适用性**

* **Level 1- Linux Host OS**

**描述**

如果可以，审计`/usr/bin/docker-runc`。

**缘由**

除了审计您的常规Linux文件系统和系统调用外，还要审计所有Docker相关的文件和目录。 Docker daemon以`“root”` 权限运行。 它的行为取决于一些关键的文件和目录。`/usr/bin/docker-runc` 就是这样一个文件。 Docker现在依靠 `containerd` 和 `runC` 来产生容器。 如果可以，它必须被审计。

**审计**

验证是否存在与 `/usr/bin/docker-runc` 文件相对应的审计规则。

例如，执行以下命令：

```text
auditctl -l | grep /usr/bin/docker-runc
```

这应该列出 `/usr/bin/docker-runc` 文件的规则。

**修正**

为 `/usr/bin/docker-runc` 文件添加一条规则。

例如， 在`/etc/audit/audit.rules`文件中添加如下行：

```text
-w /usr/bin/docker-runc -k docker
```

然后，重新启动审计daemon。 例如，

```text
service auditd restart
```

**影响**

审计会生成相当大的日志文件。 确保定期转移和归档。 此外，创建一个单独的审计分区，以避免填充根文件系统。

**默认值**

默认情况下，不会审计Docker相关的文件和目录。 文件 `/usr/bin/docker- runc` 可能在系统上不可用。 在这种情况下，这个建议是不适用的。

**参考文献**

```text
1. https://access.redhat.com/documentation/en- US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/chap-system_auditing.html
2. https://github.com/docker/docker/pull/20662
3. https://containerd.tools/
4. https://github.com/opencontainers/runc
```

