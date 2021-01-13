# 推荐六

## 安全操作

本章节介绍了Docker部署的一些操作安全方面的东西。这些是应该遵循的最佳实践。 这里的大部分建议是只是提醒组织应该延长当前的安全最佳实践和政策包括容器。

### 6.1 定期对您的主机系统和容器进行安全审计（Not Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

定期对您的主机系统和容器进行安全审计，以确定可能使您的系统受到危害的任何错误配置或漏洞。

**缘由**

对主机系统和容器执行定期和专门的安全审计，可以提供那些在日常工作中你可能不了解的深入性的安全见解，安全漏洞的鉴定会比较轻松，而且这一切都会提高你的环境的安全性。

**审计**

遵循你所在组织的安全审计策略和要求。

**修正**

遵循你所在组织的安全审计策略和要求。

**影响**

无

**默认值**

不适用

**参考文献**

```text
1. http://searchsecurity.techtarget.com/IT-security-auditing-Best-practices-for-conducting-audits
```

### 6.2 监测Docker容器的使用，性能和计量（metering）（Not Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

容器可能运行对您的业务至关重要的服务。 监测他们的使用，性能和计量将是非常重要的。

**缘由**

跟踪容器的使用、性能和和相关的计量数据，在你使用容器来运行关键服务时，这对你来说很关键，它会给带来：

* 容量管理和优化
* 性能管理
* 全面可视化

容器性能的这种深度可见性将帮助您确保容器的高可用性和最小的停机时间。

**审计**

```text
docker stats $(docker ps --quiet)
```

上述命令将为每个正在运行的容器返回CPU，内存和网络统计信息。

**修正**

使用软件或容器来跟踪容器使用情况，报告性能和计量。

**影响**

要获得容器指标，您将不得不在特权模式下使用另一个容器，或者使用可进入各种容器名称空间的软件。 对所有容器的名称空间提供不受限制的访问可能太冒险了。

**默认值**

默认情况下，对于每个容器，系统通过执行控制组（cgroups）来跟踪有关CPU，内存和块I / O使用情况的运行时度量标准，如下所示：

```text
CPU - /sys/fs/cgroup/cpu/system.slice/docker-$INSTANCE_ID.scope/ 
Memory - /sys/fs/cgroup/memory/system.slice/docker-$INSTANCE_ID.scope/ 
Block I/O - /sys/fs/cgroup/blkio/system.slice/docker-$INSTANCE_ID.scope/
```

**参考文献**

```text
1. https://docs.docker.com/articles/runmetrics/
2. https://github.com/google/cadvisor
3. https://docs.docker.com/reference/commandline/cli/#stats
```

### 6.3 备份容器数据（Not Scored）

**配置适用性**

* **Level 1- Docker**

**描述**

定期对容器的数据卷进行备份

**缘由**

容器可能运行着对你的业务至关重要的服务，定期备份数据会保证如果有任何数据的丢失的话你仍然有备份数据。而数据的真的丢失的话，很有可能毁掉你的业务。

**审计**

询问系统管理员是否定期备份容器数据卷。 验证备份的副本并确保遵循组织的备份策略。 另外，您可以对每个容器实例执行以下命令，以在容器文件系统中列出已更改的文件和目录。 理想情况下，容器的文件系统上不应该存储任何内容。

```text
docker diff $INSTANCE_ID
```

**修正**

您应该遵循组织的数据备份策略。 您可以使用`'--volumes-from'`参数备份您的容器数据卷，如下所示：

```text
$> docker run <Run arguments> --volumes-from $INSTANCE_ID -v [host-dir]:[container- dir] <Container Image Name or ID> <Command>
```

例如，

```text
docker run --volumes-from 699ee3233b96 -v /mybackup:/backup centos tar cvf /backup/backup.tar /exampledatatobackup
```

**影响**

无

**默认值**

默认情况下，容器数据卷不会进行数据备份。

**参考文献**

```text
1. http://docs.docker.com/userguide/dockervolumes/#backup-restore-or-migrate-data-volumes
2. http://stackoverflow.com/questions/26331651/back-up-docker-container-that-has-a-volume
3. http://docs.docker.com/reference/commandline/cli/#diff
```

### 6.4避免镜像蔓延（Not Scored）

**配置适用性**

* **Level 1- Linux Host OS**

**描述**

不要在同一个主机上保留大量的容器镜像。 根据需要只使用标记的镜像。

**缘由**

标记的镜像有助于从“最新”回滚到生产中特定版本。 未使用或旧标记的镜像可能包含可能被利用的漏洞（如果实例化）。 此外，如果您无法从系统中删除未使用的映像，并且存在各种冗余和未使用的映像，主机文件系统可能会变满，并可能导致拒绝服务。

**审计**

步骤1：通过执行下面命令列出所有当前实例化的镜像ID：

```text
docker images --quiet | xargs docker inspect --format '{{ .Id }}: Image={{.Config.Image }}'
```

步骤2：通过执行以下命令列出系统中存在的所有镜像：

```text
docker images
```

步骤3：比较步骤1和步骤2中的镜像ID列表，找出当前未被实例化的镜像。 如果发现任何此类未使用或旧镜像，请与系统管理员讨论是否需要将这些镜像保存在系统中。 如果不满足，不符合此基线。

**修正**

保留您实际需要的一组镜像，并建立一个工作流，以从主机中删除旧的或陈旧的镜像。 此外，使用如摘要的功能从仓库中获取特定的镜像。 另外，您可以按照下面的步骤来找出系统中未使用的镜像并将其删除。

步骤1：执行如下命令列出所有的近期实例化的镜像的ID：

```text
docker images --quiet | xargs docker inspect --format '{{ .Id }}: Image={{.Config.Image }}'
```

步骤2：执行如下命令列出系统中当前所有的镜像:

```text
docker images
```

步骤3：对比步骤1和步骤2中列出的镜像并且找出近期未实例化过得镜像。

步骤4：决定是否继续保留最近未使用的镜像，如果你想删除它们的话执行如下命令：

```text
docker rmi $IMAGE_ID
```

**影响**

无

**默认值**

在主机上，在管理员移除所有的与镜像和分层的文件系统相关的标签前，它们仍然是可用的。

**参考文献**

```text
1. http://craiccomputing.blogspot.in/2014/09/clean-up-unused-docker-containers-and.html
2. https://forums.docker.com/t/command-to-remove-all-unused-images/20/8
3. https://github.com/docker/docker/issues/9054
4. http://docs.docker.com/reference/commandline/cli/#rmi
5. http://docs.docker.com/reference/commandline/cli/#pull
6. https://github.com/docker/docker/pull/11109
```

### 6.5 避免容器蔓延（Not Scored）

**配置适用性**

* **Level 1- Linux Host OS**

**描述**

不要在同一个主机上保留大量的容器。

**缘由**

容器的灵活性使得运行多个应用程序的实例变得很容易，并间接导致存在不同安全补丁级别的Docker镜像。 这也意味着你正在消耗主机资源。 在特定的主机上有不止一个容器的可管理数量，这使得这种情况容易受到错误处理，配置错误和碎片的影响。 因此，避免容器蔓延，并将主机上的容器数量控制在一个可管理的总量上。

**审计**

执行`docker info`得到主机上的容器数量：

```text
docker info --format '{{.Containers}}'
```

现在，执行如下命令得到主机上的正在运行的或者已经停止的容器。

```text
docker info --format '{{.ContainersRunning}}'
docker info --format '{{.ContainersStopped}}'
```

如果主机上已经停止的容器数量大于正在运行的容器的数量太多（25或者更多）的话，该主机上的容器很有可能已经被蔓延了。

**修正**

定期检查每个主机的容器清单，并使用以下命令清理已停止的容器：

```text
docker container prune
```

**影响**

如果每个主机的容器数量太少，那么也许你没有充分利用你的主机资源。

**默认值**

默认情况下，Docker不会限制主机上的容器数量。

**参考文献**

```text
1. https://zeltser.com/security-risks-and-benefits-of-docker-application/
2. http://searchsdn.techtarget.com/feature/Docker-networking-How-Linux-containers-will-change-your-network
```

