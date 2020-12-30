> 根据docker官网上的介绍，docker引擎的结构图如下（[docker官网给出的结构总览](https://docs.docker.com/get-started/overview/)：）
> 引擎结构图片
> 
> Docker引擎是一个C/S结构的应用。其中扮演服务器角色的是docker daemon守护进程。扮演客户端角色的是docker CLI命令行接口，两者之间通过REST API进行通信。服务器与客户端可以运行在同一个主机上，也可以位于不同的主机上面，即可以使用docker CLI连接到一个远程主机上的Docker daemon。
> Docker CLI主要作用是与docker用户进行交互，将用户的输入转换为符合REST API的格式后，发送给docker daemon进程。然后由docker daemon进程执行相应的操作。
> 按照上图的架构（也就是官网的解释），docker daemon进程主要负责管理docker的对象，例如容器、镜像，网络和卷。不过目前为止，基于模块化的思想和解耦合的目的，docker daemon进程的功能逐渐被拆解出来并模块化了。根据《Docker Deep Dive》中的介绍，目前的dokcer daemon的主要功能有：镜像的管理和创建，提供REST API接口，身份验证和安全，网络管理和编排。主要是容器的管理被分离出来了，这样做主要是为了解耦合，降低docker daemon更新换代时的影响。 模块化后的架构可参考下图：
> 模块化后的结构图片
> 容器的相关操作在一个名为containerd的工具中进行，主要是容器的生命周期管理等，containerd也是以后台进程的方式来运行的。docker daemon与containerd之间通过gRPC进行通信。
> runc是一个独立的容器运行时工具，实质上是一个对Libcontainer进行包装了的命令行交互工具（Libcontainer在docker中的作用是取代了早期的LXC的位置），它被用来创建容器。即containerd指挥runc创建容器，然后containerd来管理容器的生命周期。
> shim进程是容器的父进程，当容器创建完毕后，相应的runc进行就会结束，shim进程就成为容器的新的父进程。它的作用有两个：<br>1.保证daemon因为某些原因宕掉后（比如更新版本时先结束daemon进程），容器不会因为管道的关闭和结束，也就是维护容器状态。<br>2.将容器的退出状态反馈给docker daemon进程。
> #### 创建容器的过程如下所述：
> **当docker daemon收到docker CLI发来的创建容器命令时，** docker daemon向containerd发出调用，containerd首先把docker镜像转换为OCI镜像（即符合开放容器计划规范的镜像），然后将OCI镜像传递给runc，接着runc与操作系统内核的API进行通信，利用Namespace、CGroup等来创建容器，容器进程作为runc的子进程启动，启动完毕后，runc退出。
> **注：** 关于REST API，gRPC，Namespace、CGroup、Libcontainer等技术或标准规范的介绍后续补充。


