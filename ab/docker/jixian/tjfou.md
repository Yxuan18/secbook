# 推荐四

## 容器镜像和构建文件

容器的基础镜像和构建文件管理着从一个特殊镜像启动容器的基本行为。确保您使用的是合适的基础镜像和恰当的构建文件，对于构建你的容器化基础服务非常重要。 为了确保您的容器化基础服务安全，您应该遵循以下关于容器基础镜像和构建文件的建议。

### 4.1 为容器创建一个用户（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

在容器镜像的Dockerfile中为容器创建一个非`root`用户。

**缘由**

如果可能，以非`root`用户运行容器是一个好习惯。 虽然现在可用使用用户名称空间映射，但如果已在容器映像中定义用户，则容器默认以该用户身份运行，并且不需要特定的用户名称空间重新映射。

**审计**

```text
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: User={{
.Config.User }}'
```

上面的命令会返回容器的用户名和用户ID。如果返回空，那么意味着容器正在以root用户运行。

**修正**

确保容器镜像的Dockerfile包含以下指令:

```text
USER <username or ID>
```

username or ID指的是可以在容器基础镜像中找到的用户。如果在容器基础镜像中没有创建特殊用户，那么在`USER`指令前添加一个`useradd`命令去添加一个特殊用户。

例如，在Dockerfile中添加下面的行，用于在容器中创建一个用户：

```text
RUN useradd -d /home/username -m -s /bin/bash username
USER username
```

**注意：如果在镜像中有容器不需要的用户，请考虐删除他们。删除这些用户后，提交镜像然后生成新的容器使用。**

**影响**

无

**默认值**

默认情况下，容器以`root`权限运行，并且在容器内部使用`root`用户身份。

**参考文献**

```text
1. https://github.com/docker/docker/issues/2918
2. https://github.com/docker/docker/pull/4572
3. https://github.com/docker/docker/issues/7906
4. https://www.altiscale.com/hadoop-blog/making-docker-work-yarn/ 
5. http://docs.docker.com/articles/security/
```

### 4.2 为容器使用可信的基础镜像（Not Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

确保容器镜像是从头开始编写，或者是基于另一个通过安全通道下载的已制定的、可信的基础镜像。

**缘由**

官方仓库是由Docker社区或者供应商发布和优化的Docker镜像。可能还有其他或许不安全的公共仓库。因此在获取容器镜像时，您需要小心谨慎。

**审计**

通过执行下面的命令来检查Docker主机：

```text
docker images
```

这将列出在当前Docker 主机上所有可用的容器镜像。咨询系统管理员，获取证明列表中的镜像都是通过安全通道从安全的源获取的证据

**修正**

配置和使用Docker信任的内容

**影响**

无

**默认值**

不适用

**参考文献**

```text
1. https://titanous.com/posts/docker-insecurity
2. https://registry.hub.docker.com/
3. http://blog.docker.com/2014/10/docker-1-3-signed-images-process-injection-
security-options-mac-shared-directories/
4. https://github.com/docker/docker/issues/8093
5. http://docs.docker.com/reference/commandline/cli/#pull
6. https://github.com/docker/docker/pull/11109
7. https://blog.docker.com/2015/11/docker-trusted-registry-1-4/
```

### 4.3 不要在容器中安装不需要的包（Not Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

容器往往是操作系统的最小精简版本。别安装任何不符合容器目的的东西。

**缘由**

用不必要的软件膨胀容器，可能会增加容器的受攻击面。这也违背了容器镜像最小精简的理念。因此，除了容器真正需要的，不要安装任何其他东西。

**审计**

步骤1：通过以下命令，列出所有运行的容器实例：

```text
docker ps --quiet
```

步骤2：对于每个容器实例，执行以，或等价的命令：

```text
docker exec $INSTANCE_ID rpm -qa
```

上面的命令会列出容器中的安装包。检查列表确保所有包的合理性。

**修正**

一开始，不要在容器上安装任何不合理的东西。如果镜像中有一些你的容器不需要的包，卸载他们。

如果可以，考虑使用最小的基础镜像而不是标准的`Redhat/Centos/Debian`镜像。例如，`BusyBox`和`Alpine`。

这不仅将你的镜像大小从150多M变为20M。还可以减少工具和路径提权途径。甚至你可以删除安装包，作为最终生产容器的加强措施。

**影响**

无

**默认值**

不适用

**参考文献**

```text
1. https://docs.docker.com/userguide/dockerimages/
2. http://www.livewyer.com/blog/2015/02/24/slimming-down-your-docker-
containers-alpine-linux
3. https://github.com/progrium/busybox
```

### 4.4 扫描、重新构建镜像，以包含安全补丁（Not Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

镜像应该被经常扫描，以确保没有任何漏洞。重新构建包含安全补丁的镜像，然后实例化新容器。

**缘由**

漏洞是可以利用的bugs，安全补丁是为了消除漏洞而进行的升级。我们可以使用镜像漏洞扫描工具找到镜像中的各种漏洞，然后查找可用的安全补丁消除这些漏洞。补丁程序将系统更新到最新的代码库。 在当前的基础代码上是很重要的，因为这是厂商集中解决问题的地方。 在应用之前评估安全补丁并遵循补丁最佳实践。

另外，如果镜像漏洞扫描工具能够执行二进制级别分析或基于哈希的验证，而不是版本字符串匹配，效果会更好。

**审计**

步骤1：执行下面的命令，列出所有运行的容器：

```text
docker ps --quiet
```

步骤2：为每一个容器实例，执行下面，或等价的命令，列出容器中的安装包。确保漏洞的安全补丁包已被安装。

```text
 docker exec $INSTANCE_ID rpm -qa
```

或者，您可以在您的系统中运行能够扫描所有镜像的镜像漏洞扫描工具，然后为检测到的漏洞应用基于修复管理程序的补丁。

**修正**

按照以下步骤，使用安全补丁重新构建镜像：

步骤1：`‘docker pull’`所有的基础镜像（也就是在Dockerfile集合中，提取在`‘FROM’`指令中声明的所有镜像，并且重新拉取他们以检测更新/补丁版本）。也可在镜像中打补丁。

步骤2：使用`'docker build --no-cache'`强制重新构建每一个镜像。

步骤3：使用升级后的镜像重新开始所有的容器。

你也可以在Dockerfile中使用`ONBUILD`指令去引发镜像更新指令，常用于知道的会被经常用作基础镜像的镜像。

**影响**

无

**默认值**

默认情况下，容器和镜像是不会自己更新的。

**参考文献**

```text
1. https://docs.docker.com/userguide/dockerimages/
2. https://docs.docker.com/docker-cloud/builds/image-scan/
3. https://blog.docker.com/2016/05/docker-security-scanning/ 
4. https://docs.docker.com/engine/reference/builder/#/onbuild
```

### 4.5 为Docker启用 Content trust \(Scored\)

**配置适用性**

* **Level 2- Docker**

**描述**

`Content trust`默认是不可用的，您应该启用它。

**缘由**

`Content trust`提供从远程Docker仓库发送和接受数据时的数字签名能力。这些签名允许客户端验证特定镜像标签的完整性和发布者。这可以确保容器镜像的来源。

**审计**

```text
echo $DOCKER_CONTENT_TRUST
```

这应该返回1。

**修正**

在bash shell中启用 `content trust`,输入下面的命令：

```text
export DOCKER_CONTENT_TRUST=1
```

或者，在您的profile文件中设置环境变量，这样每次登录都会启用`content trust`。

**影响**

在一个设置了`DOCKER_CONTENT_TRUST`的环境中，在以`-buile,create,pull,pus和run`操作镜像时，你需要遵循信任流程。您可以使用`--disable-content-trust`标志在没有`content trust`标签的镜像上运行单独的操作，但是这违背了启用`content trust`的目的，因此应尽可能避免。

**注意：`Content trust` 当前只能在Docker Hub 的用户上使用。在Docker Trusted Registry或者私有仓库上还不可用。**

**默认值**

默认，没有启用`content trust`。

**参考文献**

```text
1. https://docs.docker.com/engine/security/trust/content_trust/
2. https://docs.docker.com/engine/reference/commandline/cli/#notary
3. https://docs.docker.com/engine/reference/commandline/cli/#environment-
variables
```

### 4.6 为容器镜像添加HEALTHCHECK指令（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

在您的docker容器镜像中添加`HEALTHCHECK`指令，完对运行中容器的健康检查。

**缘由**

安全三要素中，其中一个重要的是可用性。 将`HEALTHCHECK`指令添加到您的容器镜像，确保docker引擎定期检查正在运行的针对该指令的容器实例仍在工作。

基于健康状态记录，docker引擎可以让停止工作的容器退出，并且实例化一个新的。

**审计**

运行以下命令，确保docker镜像已经设置合适的`HEALTHCHECK`指令。

```text
docker inspect --format='{{ .Config.Healthcheck }}' <IMAGE>
```

**修正**

按照Docker文档，使用`HEALTHCHECK`指令重新构建您的容器镜像。

**影响**

无

**默认值**

默认情况下，没有设置`HEALTHCHECK`。

**参考文献**

```text
1. https://github.com/docker/docker/pull/22719
```

### 4.7 不要在Dockerfile中单独使用升级指令（Not Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

在 Dockerfile中不要在一行、或者单独的使用更新指令，例如， `apt-get update` 。

**缘由**

在Dockerfile中使用单行的更新指令将会缓存更新层。因此，以后使用相同的指令构建任何镜像时，先前的缓存更新层将被使用。 这可能会在以后的版本中拒绝新的更新。

**审计**

步骤1：运行以下的命令，获取镜像列表：

```text
docker images
```

步骤2：对上面的列表中每个镜像运行下面的命令，并查找任何在单行中的更新指令:

```text
docker history <Image_ID>
```

或者，如果您可以访问该镜像的Dockerfile，请验证是否有如上描述的更新指令。

**修正**

在安装时，使用更新指令，后面带上安装说明（或任何其他）和固定版本。 这将破坏缓存并强制提取 需要的版本。 或者，您可以在docker构建过程中使用`--no-cache`标志来避免使用缓存层。

**影响**

无

**默认值**

默认情况下，docker不会强制限制使用更新指令。

**参考文献**

```text
1. https://docs.docker.com/engine/userguide/eng-image/dockerfile_best- practices/#/apt-get
2. https://github.com/docker/docker/issues/3313
```

### 4.8 删除镜像中的setuid和setgid权限（Not Scored）

**配置适用性**

* **Level 2- Docker**

**描述**

删除镜像中的`setuid`和`setgid`权限可以预防容器中的提权攻击。

**缘由**

`setuid`和`setgid`权限可以用于提升权限。虽然这些权限有时是合法需求，但是这些权限可能会被提权攻击利用。因此，你应该考虑在不需要权限的镜像中删除这些包的权限。

**审计**

在镜像上运行以下命令，列出所有具有`setuid`和`setgid`权限的可执行文件。

```text
docker run <Image_ID> find / -perm +6000 -type f -exec ls -ld {} \; 2> /dev/null
```

仔细查看列表，确保它都是合法的。

**修正**

仅在需要它们的可执行文件上允许`setuid`和`setgid`权限。 您可以通过在您的Dockerfile中加入下面的命令，最好靠近Dockerfile的末尾，以在编译时删除这些权限，：

```text
RUN find / -perm +6000 -type f -exec chmod a-s {} \; || true
```

**影响**

以上命令会破坏所有依赖`setuid`或`setgid`权限的可执行文件,包括合法的。 因此，要小心修改命令以适合您的要求，以便不会丢失合法程序的权限。 这个需要仔细检查每个可执行文件并对权限进行微调。

**默认值**

不适用

**参考文献**

```text
1. http://www.oreilly.com/webops-perf/free/files/docker-security.pdf
2. http://container-
solutions.com/content/uploads/2015/06/15.06.15_DockerCheatSheet_A2.pdf
3. http://man7.org/linux/man-pages/man2/setuid.2.html
4. http://man7.org/linux/man-pages/man2/setgid.2.html
```

### 4.9 在Dockerfile中使用COPY代替ADD（Not Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

在Dockerfile中，使用 `COPY`指令代替 `ADD`指令。

**缘由**

`COPY`指令只是将本地主机上的文件复制到容器文件系统中。 `ADD`指令可能会从远程URL检索文件并执行操作，如拆包。 因此，`ADD`指令会引入风险，诸如从URL添加没有扫描和解包分析处理的恶意文件。

**审计**

步骤1：运行以下命令，获取镜像列表

```text
docker images
```

步骤2：对上面列表的每一个镜像运行下面的命令，来查找 `ADD`指令

```text
docker history <Image_ID>
```

或者，如果您可以访问该映像的Dockerfile，请验证没有`ADD`指令。

**修正**

在Dockerfiles中使用`COPY`指令。

**影响**

你需要注意`ADD`指令提供的功能，例如从远程URL获取文件。

**默认值**

不适用

**参考文献**

```text
1. https://docs.docker.com/engine/userguide/eng-image/dockerfile_best- practices/#/add-or-copy
```

### 4.10 在Dockerfile中不要存储机密（Not Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

在Dockerfiles中不要储存任何秘密。

**缘由**

通过使用本地Docker命令，例如，docker history，各种工具和实用程序等可以轻松回溯Dockerfiles。 此外，作为一种普遍的做法，镜像发布者提供Dockerfiles来构建镜像的可信度。 因此，这些Dockerfiles中的秘密可能很容易被暴露，并可能被利用。

**审计**

步骤1：运行下面的命令，获取镜像列表

```text
docker images
```

步骤2：对上面列表中的每个镜像运行以下命令，查找可能的秘密：

```text
docker history <Image_ID>
```

或者，如果你可以访问镜像的Dockerfile,请确认没有如上所述的秘密。

**修正**

不要在Dockerfiles中存储任何类型的秘密。

**影响**

您需要找到一种方法来处理Docker镜像的秘密。

**默认值**

默认情况下，在Dockerfiles中不限制存储配置机密。

**参考文献**

```text
1. https://github.com/docker/docker/issues/13490
2. http://12factor.net/config
3. https://avicoder.me/2016/07/22/Twitter-Vine-Source-code-dump/
```

### 4.11只安装已验证的包（Not Scored）

**配置适用性**

* **Level 2- Docker**

**描述**

在镜像中安装包之前，证实它的可靠性。

**缘由**

验证软件包的真实性对于构建安全的容器镜像至关重要。篡改过的软件包可能是恶意的，或者有一些已知的漏洞可以被利用。

**审计**

步骤1：运行下面的命令，获取镜像列表

```text
docker images
```

步骤2：对上面列表中的每个镜像运行以下命令，并寻找方法确定安装包的真实性。 可以通过使用GPG密钥或其他安全包分发机制：

```text
docker history <Image_ID>
```

或者，如果您可以访问该镜像的Dockerfile，请验证被检查包的可靠性。

**修正**

使用GPG密钥下载和验证软件包，或者其他你选择的安全包分配机制。

**影响**

无

**默认值**

不适用

**参考文献**

```text
1. http://www.oreilly.com/webops-perf/free/files/docker-security.pdf
2. https://github.com/docker-
library/httpd/blob/12bf8c8883340c98b3988a7bade8ef2d0d6dcf8a/2.4/Dockerfile
3. https://github.com/docker-library/php/blob/d8a4ccf4d620ec866d5b42335b699742df08c5f0/7.0/alpine/Dockerfile
4. https://access.redhat.com/security/team/key
```

