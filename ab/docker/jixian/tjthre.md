# 推荐三

## docker daemon 配置文件

本节涵盖了Docker相关文件和目录的权限和所有权。 保持可能包含敏感参数的文件和目录安全，对于Docker daemon 的正确和安全运行非常重要。

### 3.1 验证docker.service文件的所有权设置为root：root（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证`“docker.service”`文件所有权和组所有权已正确设置为`“root”`。

**缘由**

`“docker.service”`文件包含了一些可能会改变Docker daemon行为的敏感参数。因此，它应该由`“root”`用户和`“root”` 用户组拥有，以此来保证文件的完整性。

**审计**

步骤1：找出文件位置

```text
systemctl show -p FragmentPath docker.service
```

步骤2：如果文件不存在，则此基线不适用。如果该文件存在，请使用正确的文件路径执行以下命令，以验证该文件是否由`“root”`用户和`“root”`用户组拥有

例如：

```text
stat -c %U:%G /usr/lib/systemd/system/docker.service | grep -v root:root
```

正常情况上述命令应该没有任何返回

**修正**

步骤1：找出文件位置：

```text
systemctl show -p FragmentPath docker.service
```

步骤2：如果文件不存在，则此基线不适用。如果该文件存在，请使用正确的文件路径执行以下命令，将文件的用户和用户组设置为`“root”`

例如：

```text
chown root:root /usr/lib/systemd/system/docker.service
```

**影响**

无

**默认值**

该文件可能不存在于系统上。在这种情况下，此基线不适用。默认情况下，如果文件存在，则该文件的用户和用户组应被正确的设置为`“root”`。

**参考文献**

```text
1. https://docs.docker.com/engine/admin/systemd/
```

### 3.2 验证docker.service文件权限是否被设置为644或更严格（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证`“docker.service”`文件的权限是否正确的设置为`“644”`或更严格

**缘由**

`“docker.service”`文件包含一些可能改变Docker daemon行为的敏感参数。因此除了`“root”`以外，其他任何用户的写入操作都是不被允许的，以此来保持文件的完整性。

**审计**

步骤1：找出文件位置

```text
systemctl show -p FragmentPath docker.service
```

步骤2：如果文件不存在，则此基线不适用。如果该文件存在，请使用正确的文件路径执行以下命令，以验证文件权限是否被设置为`“644”`或更严格

例如，

```text
stat -c %a /usr/lib/systemd/system/docker.service
```

**修正**

步骤1：找出文件位置

```text
systemctl show -p FragmentPath docker.service
```

步骤2：如果文件不存在，则此基线不适用。如果该文件存在，请使用正确的文件路径执行以下命令，将文件权限设置为`“644”`

例如，

```text
chmod 644 /usr/lib/systemd/system/docker.service
```

**影响**

无

**默认值**

该文件可能不存在于系统上。在这种情况下，此基线不适用。默认情况下，如果文件存在，则该文件权限应被正确设置为`“644”`。

**参考文献**

```text
1. https://docs.docker.com/articles/systemd/
```

### 3.3 验证docker.socket文件的所有权设置为root：root（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证`“docker.socket`”文件的所有权和组所有权是否正确设置为`“root”`。

**缘由**

`“docker.socket”`文件包含一些可能会改变Docker远程API行为的敏感参数。因此，它应该由`“root”`用户和用户组所有，以此来保持文件的完整性。

**审计**

步骤1：找出文件位置：

```text
systemctl show -p FragmentPath docker.socket
```

步骤2：如果文件不存在，则此基线不适用。如果该文件存在，请使用正确的文件路径执行以下命令，以验证该文件是否由`“root”`用户和`“root”`用户组拥有

例如：

```text
stat -c %U:%G /usr/lib/systemd/system/docker.socket | grep -v root:root
```

正常情况上述命令应该没有任何返回。

**修正**

步骤1：找出文件位置

```text
systemctl show -p FragmentPath docker.socket
```

步骤2：如果文件不存在，则此基线不适用。如果该文件存在，请使用正确的文件路径执行以下命令将文件的用户和用户组设置为`“root”`

例如，

```text
chown root:root /usr/lib/systemd/system/docker.socket
```

**影响**

无

**默认值**

该文件可能不存在于系统上。在这种情况下，此基线不适用。默认情况下，如果文件存在，则该文件的用户和用户组应被正确的设置为`“root”`。

**参考文献**

```text
1. https://docs.docker.com/articles/basics/#bind-docker-to-another-hostport-or-a- unix-socket
2. https://github.com/YungSang/fedora-atomic- packer/blob/master/oem/docker.socket
3. http://daviddaeschler.com/2014/12/14/centos-7rhel-7-and-docker-containers- on-boot/
```

### 3.4 验证docker.socket文件权限是否被设置为644或更严格（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证`“docker.socket”`文件的权限是否正确的设置为`“644”`或更严格

**缘由**

`“docker.socket”`文件包含一些可能改变Docker远程API行为的敏感参数。因此应该只有`“root”` 用户才能对其进行写入操作，以此来保持文件的完整性

**审计**

步骤1：找出文件位置

```text
systemctl show -p FragmentPath docker.socket
```

步骤2：如果文件不存在，则此基线不适用。如果该文件存在，请使用正确的文件路径执行以下命令，以验证文件权限是否被设置为`“644”`或更严格

例如，

```text
stat -c %a /usr/lib/systemd/system/docker.socket
```

**修正**

步骤1：找出文件位置

```text
systemctl show -p FragmentPath docker.socket
```

步骤2：如果文件不存在，则此基线不适用。如果该文件存在，请使用正确的文件路径执行以下命令，将文件权限设置为`“644”`

例如，

```text
chmod 644 /usr/lib/systemd/system/docker.socket
```

**影响**

无

**默认值**

该文件可能不存在于系统上。在这种情况下，此基线不适用。默认情况下，如果文件存在，则该文件权限应被正确设置为`“644”`。

**参考文献**

```text
1. https://docs.docker.com/articles/basics/#bind-docker-to-another-hostport-or-a- unix-socket
2. https://github.com/YungSang/fedora-atomic- packer/blob/master/oem/docker.socket
3. http://daviddaeschler.com/2014/12/14/centos-7rhel-7-and-docker-containers- on-boot/
```

### 3.5 验证/etc/docker目录的所有权是否设置为root：root（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证`“/etc/docker“`目录的所有权和组所有权是否正确设置为`“root”`。

**缘由**

除了各种敏感文件，`“/etc/docker”`目录还包含证书和密钥。因此，它应该由`“root”`用户和用户组拥有，以此来保持目录的完整性。

**审计**

执行以下命令来验证该目录是否由`“root”`用户和用户组拥有

```text
stat -c %U:%G /etc/docker | grep -v root:root
```

正常情况上述命令应该没有任何返回。

**修正**

```text
chown root:root /etc/docker
```

这会将该目录的用户和用户组所有权设置为`“root”`

**影响**

无

**默认值**

默认情况下，此目录的用户和用户组所有权已正确设置为`“root”`

**参考文献**

```text
1. https://docs.docker.com/articles/certificates/
```

### 3.6 验证/etc/docker目录权限是否被设置为755或更严格（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证`/etc/docker`目录权限是否被正确设置为`'755'`或更严格

**缘由**

除了各种敏感文件，`“/etc/docker”`目录还包含证书和密钥。因此只有`“root”`用户才能进行写入操作，以此来保持目录的完整性

**审计**

执行以下命令以验证目录权限是否为`“755”`或更严格

```text
stat -c %a /etc/docker
```

**修正**

```text
chmod 755 /etc/docker
```

这将设置目录的权限为`'755'`

**影响**

无

**默认值**

默认情况下，此目录的权限已正确设置为“755”

**参考文献**

```text
1. https://docs.docker.com/articles/certificates/
```

### 3.7 验证仓库证书文件所有权是否设置为root：root（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证所有仓库证书文件（通常在`/etc/docker/certs.d/<registry-name>`目录下）是否由`"root"`用户和用户组拥有

**缘由**

`/etc/docker/certs.d/ <registry-name>`目录包含Docker 仓库的证书。这些证书文件必须由`“root”`用户和用户组拥有，以维护证书的完整性

**审计**

执行以下命令以验证仓库证书文件是否由`"root"`用户和用户组拥有

```text
stat -c %U:%G /etc/docker/certs.d/* | grep -v root:root
```

正常情况上述命令应该没有任何返回。

**修正**

```text
chown root:root /etc/docker/certs.d/<registry-name>/*
```

这会将仓库证书文件的用户和用户组所有权设置为`“root”`

**影响**

无

**默认值**

默认情况下，仓库证书文件的用户和用户组所有权已正确设置为`“root”`

**参考文献**

```text
1. https://docs.docker.com/articles/certificates/
2. http://docs.docker.com/reference/commandline/cli/#insecure-registries
```

### 3.8 验证仓库证书文件权限是否设置为444或更严格（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证所有仓库证书文件（通常在`/etc/docker/certs.d/<registry-name>`目录下）的权限是否是`"444"`或更严格。

**缘由**

`/etc/docker/certs.d/<registry-name>`目录包含了Docker 仓库证书。这些证书文件必须具有`“444”`权限，以此维护证书的完整性

**审计**

执行以下命令以验证仓库证书文件权限为`“444”`或更严格

```text
stat -c %a /etc/docker/certs.d/<registry-name>/*
```

**修正**

```text
chmod 444 /etc/docker/certs.d/<registry-name>/*
```

这会将仓库证书文件的所有权设置为`“444”`

**影响**

无

**默认值**

默认情况下，仓库证书文件的权限可能不是`“444”`。默认文件权限由系统或用户特定的 `umask`值进行管理

**参考文献**

```text
1. https://docs.docker.com/articles/certificates/
2. http://docs.docker.com/reference/commandline/cli/#insecure-registries
```

### 3.9 验证TLS CA证书文件的所有权是否设置为root：root（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证TLS CA证书文件（与`“--tlscacert”`参数一起传递的文件）是否由`"root"`用户和用户组拥有

**缘由**

TLS CA证书文件应该被保护，以免受任何篡改。它通过给定的CA证书对Docker服务进行身份验证。因此，它必须由“`"root"`用户和用户组拥有，以保持CA证书的完整性

**审计**

执行以下命令以验证TLS CA证书文件是否由`"root"`用户和用户组拥有

```text
stat -c %U:%G <path to TLS CA certificate file> | grep -v root:root
```

正常情况上述命令应该没有任何返回。

**修正**

```text
chown root:root <path to TLS CA certificate file>
```

这会将TLS CA证书文件的用户和用户组所有权设置为`"root"`

**影响**

无

**默认值**

默认情况下，TLS CA证书文件的用户和用户组所有权已正确设置为`“root”`

**参考文献**

```text
1. https://docs.docker.com/articles/certificates/ 
2. http://docs.docker.com/articles/https/
```

### 3.10 验证TLS CA证书文件权限是否设置为444或更严格（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证TLS CA证书文件（与`“ -- tlscacert”`参数一起传递的文件）的权限是否是`"444"`或更严格。

**缘由**

TLS CA证书文件应该被保护，以免受任何篡改。它通过给定的CA证书对Docker服务进行身份验证。因此，它必须具有`“444”`权限，以保持CA证书的完整性

**审计**

执行以下命令以验证TLS CA文件权限为`“444”`或更严格

```text
stat -c %a <path to TLS CA certificate file>
```

**修正**

```text
chmod 444 <path to TLS CA certificate file>
```

这会将TLS CA文件的权限设置为`“444”`

**影响**

无

**默认值**

默认情况下，TLS CA文件的权限可能不是`“444”`。默认文件权限由系统或用户特定的`umask`值进行管理

**参考文献**

```text
1. https://docs.docker.com/articles/certificates/ 
2. http://docs.docker.com/articles/https/
```

### 3.11 验证Docker服务证书文件的所有权是否设置为root：root（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证Docker服务证书文件（与`“--tlscert”`参数一起传递的文件）是否由`“root”`用户和用户组拥有

**缘由**

Docker服务证书文件应该被保护，以免受任何篡改。它通过给定的服务证书对Docker服务进行身份验证。因此，它必须由“`"root"`用户和用户组拥有，以保持证书的完整性

**审计**

执行以下命令以验证Docker服务证书文件是否由`"root"`用户和用户组拥有

```text
stat -c %U:%G <path to Docker server certificate file> | grep -v root:root
```

正常情况上述命令应该没有任何返回。

**修正**

```text
chown root:root <path to Docker server certificate file>
```

这会将Docker服务证书文件的用户和用户组所有权设置为`“root”`

**影响**

无

**默认值**

默认情况下，Docker服务证书文件的用户和用户组所有权已正确设置为`“root”`

**参考文献**

```text
1. https://docs.docker.com/articles/certificates/ 
2. http://docs.docker.com/articles/https/
```

### 3.12 验证Docker服务证书文件权限是否设置为444或更严格（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证Docker服务证书文件（与`“--tlscert”`参数一起传递的文件）的权限是否是`"444"`或更严格

**缘由**

Docker服务证书文件应该被保护，以免受任何篡改。它通过给定的服务证书对Docker服务进行身份验证。因此，它必须具有`“444”`权限，以保持Docker服务证书的完整性

**审计**

执行以下命令以验证Docker服务证书文件权限为`“444”`或更严格

```text
stat -c %a <path to Docker server certificate file>
```

**修正**

```text
chmod 444 <path to Docker server certificate file>
```

这会将Docker服务证书文件的权限设置为`“444”`

**影响**

无

**默认值**

默认情况下，Docker服务证书文件的权限可能不是`“444”`。默认文件权限由系统或用户特定的`umask`值进行管理

**参考文献**

```text
1. https://docs.docker.com/articles/certificates/ 
2. http://docs.docker.com/articles/https/
```

### 3.13 验证Docker服务证书密钥文件的所有权是否设置为root：root（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证Docker服务证书密钥文件（与`“--tlskey”`参数一起传递的文件）是否由`“root”`用户和用户组拥有

**缘由**

Docker服务证书密钥文件应被保护，以防止被篡改以及任何不必要的读取。它拥有Docker服务证书的私钥。因此，它必须由`“root”`用户和用户组拥有，以保持证书的完整性

**审计**

执行以下命令以验证Docker服务证书密钥文件是否由`"root"`用户和用户组拥有

```text
stat -c %U:%G <path to Docker server certificate key file> | grep -v root:root
```

正常情况上述命令应该没有任何返回。

**修正**

```text
chown root:root <path to Docker server certificate key file>
```

这会将Docker服务证书密钥文件的用户和用户组所有权设置为`“root”`

**影响**

无

**默认值**

默认情况下，Docker服务证书密钥文件用户和用户组的所有权已正确设置为`“root”`

**参考文献**

```text
1. https://docs.docker.com/articles/certificates/ 
2. http://docs.docker.com/articles/https/
```

### 3.14 验证Docker服务证书密钥文件权限是否设置为400（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证Docker服务证书密钥文件（与`“--tlskey”`参数一起传递的文件）的权限是否是`"400"`

**缘由**

Docker服务证书密钥文件应被保护，以防止任何篡改及不必要的读取。它拥有Docker服务器证书的私钥。因此必须具有`“400”`的权限，以保持Docker服务证书的完整性

**审计**

执行以下命令来验证Docker服务证书密钥文件的权限是否为`“400”`

```text
stat -c %a <path to Docker server certificate key file>
```

**修正**

```text
chmod 400 <path to Docker server certificate key file>
```

这会将Docker服务证书密钥文件权限设置为`“400”`

**影响**

无

**默认值**

默认情况下，Docker服务证书密钥文件的权限可能不是`“400”`。默认文件权限由系统或用户特定的`umask`值进行管理

**参考文献**

```text
1. https://docs.docker.com/articles/certificates/ 
2. http://docs.docker.com/articles/https/
```

### 3.15 验证Docker套接字文件所有权是否设置为root：docker（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证Docker套接字文件是由`“root”`用户拥有，且被`“docker”`用户组拥有

**缘由**

Docker daemon以`“root”`身份运行。因此，默认的Unix套接字必须由`“root”`用户拥有。如果任何其他用户或进程拥有此套接字，那么该非特权用户或进程就可能与Docker daemon 进行交互。而且，这样的非特权用户或进程还可能与容器交互。这既不安全也是我们不希望的行为。 此外，Docker安装程序创建一个名为`“docker”`的Unix组。您可以将用户添加到该组，这些用户将能够读写默认的Docker Unix套接字。`“docker”`组的成员资格由系统管理员严格控制。如果其他任何组拥有此套接字，那么该组的成员可能会与Docker daemon 进行交互。并且，这样的用户组可能不像`“docker”`组那样受到严格控制。这既不安全也是我们不希望看到的行为。 因此，默认的Docker Unix套接字文件必须由`“root”`用户拥有，并由`“docker”`用户组拥有，以保持套接字文件的完整性

**审计**

执行以下命令验证Docker套接字文件是由`“root”`用户拥有，且由`“docker”`用户组拥有

```text
stat -c %U:%G /var/run/docker.sock | grep -v root:docker
```

正常情况上述命令应该没有任何返回。

**修正**

```text
chown root:docker /var/run/docker.sock
```

这会将默认的Docker套接字文件的所有权和组所有权分别设置为`“root”`和`“docker”`

**影响**

无

**默认值**

默认情况下，Docker套接字文件的所有权和组所有权已被正确设置为`“root：docker”`

**参考文献**

```text
1. https://docs.docker.com/reference/commandline/cli/#daemon-socket-option
2. https://docs.docker.com/articles/basics/#bind-docker-to-another-hostport-or-a-
unix-socket
```

### 3.16 验证Docker套接字文件权限是否设置为660或更严格（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证Docker套接字文件权限是否为`“660”`或更严格

**缘由**

只允许`“root”`和`“docker”`组的成员读写默认的Docker Unix套接字。因此，Docket套接字文件权限是否为`“660”`或更严格

**审计**

执行以下命令以验证Docker套接字文件权限是否为`“660”`或更严格

```text
stat -c %a /var/run/docker.sock
```

**修正**

```text
chmod 660 /var/run/docker.sock
```

这会将Docker套接字文件的文件权限设置为`“660”`

**影响**

无

**默认值**

默认情况下，Docker套接字文件的权限已正确设置为`“660”`

**参考文献**

```text
1. https://docs.docker.com/reference/commandline/cli/#daemon-socket-option
2. https://docs.docker.com/articles/basics/#bind-docker-to-another-hostport-or-a-
unix-socket
```

### 3.17 验证daemon.json文件的所有权是否设置为root：root（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证`“daemon.json”`文件是否由`“root”`用户和用户组拥有

**缘由**

`"daemon.json"`文件包含可能改变docker daemon行为的敏感参数。因此，它应该由`“root”`用户和用户组拥有，以保持文件的完整性

**审计**

执行以下命令以验证此文件是否由`"root"`用户和用户组拥有

```text
stat -c %U:%G /etc/docker/daemon.json | grep -v root:root
```

正常情况上述命令应该没有任何返回。

**修正**

```text
chown root:root /etc/docker/daemon.json
```

这会将此文件的用户和用户组所有权设置为`“root”`

**影响**

无

**默认值**

该文件可能不存在于系统上。 在这种情况下此基线不适用

**参考文献**

```text
1. https://docs.docker.com/engine/reference/commandline/daemon/#daemon- configuration-file
```

### 3.18 验证daemon.json文件权限是否被设置为644或更严格（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证`“daemon.json”`文件权限是否正确设置为`“644”`或更严格

**缘由**

`"daemon.json"`文件包含可能改变docker daemon行为的敏感参数.因此，它应该由`“root”`用户和用户组拥有，以保持文件的完整性

**审计**

执行以下命令以验证此文件权限是否被正确设置为`“644”`或更严格

```text
stat -c %a /etc/docker/daemon.json
```

**修正**

```text
chmod 644 /etc/docker/daemon.json
```

这会将该文件的文件权限设置为`“644”`

**影响**

无

**默认值**

该文件可能不存在于系统上。 在这种情况下此基线不适用

**参考文献**

```text
1. https://docs.docker.com/engine/reference/commandline/daemon/#daemon- configuration-file
```

### 3.19 验证/etc/default/docker文件的所有权是否设置为root：root（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证`“/etc/default/docker”`文件是否由`“root”`用户和用户组拥有

**缘由**

`”/etc/default/docker“`文件包含可能会改变docker daemon行为的敏感参数。因此，它应该由`“root”`用户和用户组拥有，以保持目录的完整性

**审计**

执行以下命令以验证此文件是否由`"root"`用户和用户组拥有

```text
stat -c %U:%G /etc/default/docker | grep -v root:root
```

正常情况上述命令应该没有任何返回。

**修正**

```text
chown root:root /etc/default/docker
```

这会将此文件的用户和用户组所有权设置为`“root”`

**影响**

无

**默认值**

该文件可能不存在于系统上。 在这种情况下此基线不适用

**参考文献**

```text
1. https://docs.docker.com/engine/admin/configuring/
```

### 3.20 验证/etc/default/docker文件权限是否被设置为644或更严格（Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

验证`/etc/default/docker`文件权限是否正确设置为`“644”`或更严格

**缘由**

`/etc/default/docker`文件包含可能改变docker daemon行为的敏感参数。因此，它应该由`“root”`用户和用户组拥有，以保持文件的完整性

**审计**

执行以下命令以验证此文件权限是否被正确设置为`“644”`或更严格

```text
stat -c %a /etc/default/docker
```

**修正**

```text
chmod 644 /etc/default/docker
```

这会将该文件的文件权限设置为`“644”`

**影响**

无

**默认值**

该文件可能不存在于系统上。 在这种情况下此基线不适用

**参考文献**

```text
1. https://docs.docker.com/engine/admin/configuring/
```

