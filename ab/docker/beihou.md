# 基础命令的背后

### 1、docker pull

1. Docker Client处理用户发起的docker pull命令，解析完请求以及参数之后，发送一个HTTP请求给Docker Server，HTTP请求方法为POST，请求URL为"/images/create？"+"xxx"，实际意义为下载相应的镜像。
2. Docker Server接收以上HTTP请求，并交给mux.Router，mux.Router通过URL以及请求方法类型来确定执行该请求的具体handler。
3. mux.Router将请求路由分发至相应的handler，具体为PostImagesCreate。
4. 在PostImageCreate这个handler之中，创建并初始化一个名为"pull"的Job，之后触发执行该Job。
5. 名为"pull"的Job在执行过程中执行pullRepository操作，即从Docker Registry中下载相应的一个或者多个Docker镜像。
6. 名为"pull"的Job将下载的Docker镜像交给graphdriver管理。
7. graphdriver负责存储Docker镜像，一方面将实际镜像存储至本地文件系统中，另一方面为镜像创建对象，由DockerDaemon统一管理。

### 2、docker run

1. Docker Client处理用户发起的docker run命令，解析完请求与参数之后，向Docker Server发送一个HTTP请求，HTTP请求方法为POST，请求URL为"/containers/create？"+"xxx"，实际意义为创建一个容器对象，即Docker Daemon程序逻辑中的容器对象，并非实际运行的容器。
2. Docker Server接收以上HTTP请求，并交给mux.Router，mux.Router通过URL以及请求方法来确定执行该请求的具体handler。
3. mux.Router将请求路由分发至相应的handler，具体为PostContainersCreate。
4. 在PostContainersCreate这个handler之中，创建并初始化一个名为"create"的Job，之后触发执行该Job。
5. 名为"create"的Job在运行过程中执行Container.Create操作，该操作需要获取容器镜像来为Docker容器准备rootfs，通过graphdriver完成。
6. graphdriver从Graph中获取创建Docker容器rootfs所需要的所有镜像。
7. graphdriver将rootfs的所有镜像通过某种联合文件系统的方式加载至Docker容器指定的文件目录下。 
8. 若以上操作全部正常执行，没有返回错误或异常，则Docker Client收到Docker Server返回状态之后，发起第二次HTTP请求。请求方法为"POST"，请求URL为"/containers/"+container\_ID+"/start"，实际意义为启动时才创建完毕的容器对象，实现物理容器的真正运行。
9. Docker Server接收以上HTTP请求，并交给mux.Router，mux.Router通过URL以及请求方法来确定执行该请求的具体handler。
10. mux.Router将请求路由分发至相应的handler，具体为PostContainersStart。
11. 在PostContainersStart这个handler之中，创建并初始化名为"start"的Job，之后触发执行该Job。
12. 名为"start"的Job执行需要完成一系列与Docker容器相关的配置工作，其中之一是为Docker容器网络环境分配网络资源，如IP资源等，通过调用networkdriver完成。 
13. networkdriver为指定的Docker容器分配网络资源，其中有IP、port等，另外为容器设置防火墙规则。
14. 返回名为"start"的Job，执行完一些辅助性操作后，Job开始执行用户指令，调用execdriver。
15. execdriver被调用，开始初始化Docker容器内部的运行环境，如命名空间、资源控制与隔离，以及用户命令的执行，相应的操作转交至libcontainer来完成。
16. libcontainer被调用，完成Docker容器内部的运行环境初始化，并最终执行用户要求启动的命令。



