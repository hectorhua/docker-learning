## Get Started, Part 1: Orientation and setup 介绍与安装

欢迎学习Docker，本节作为Docker入门教程包含以下内容：  

1. Set up your Docker environment 安装Docker运行环境
2. Build an image and run it as one container 构建镜像并以容器方式启动运行
3. Scale your app to run multiple containers 扩展应用运行到多个容器副本
4. Distribute your app across a cluster 在Docker集群中分发应用
5. Stack services by adding a backend database 使用stack将后端数据库集成到应用服务
6. Deploy your app to production 部署应用并发布

### Docker concepts 概念
Docker is a platform for developers and sysadmins to develop, deploy, and run applications with containers. The use of Linux containers to deploy applications is called containerization. Containers are not new, but their use for easily deploying applications is.

> Docker是一个适于开发人员和系统管理人员运用容器进行开发，部署和运行程序的平台。使用Linux容器部署应用的过程称为容器化。容器技术并不是新技术，但使用它们轻松部署应用程序是新兴的技术潮流。

Containerization is increasingly popular because containers are:

* Flexible: Even the most complex applications can be containerized.
* Lightweight: Containers leverage and share the host kernel.
* Interchangeable: You can deploy updates and upgrades on-the-fly.
* Portable: You can build locally, deploy to the cloud, and run anywhere.
* Scalable: You can increase and automatically distribute container replicas.
* Stackable: You can stack services vertically and on-the-fly.

> 容器化技术的流行是由于容器具有下列的特性：  
> * 更灵活：即使是最复杂的应用也可以被容器化  
> * 更轻量：容器共享并最大化的利用了主机的内核  
> * 热更新：可以实时的更新和升级  
> * 易迁移：一次构建，随处部署  
> * 易伸缩：可实时增加并自动分发容器副本  
> * 易叠加：可实时在垂直方向叠加服务

### Images and containers 镜像与容器
A container is launched by running an image. An image is an executable package that includes everything needed to run an application--the code, a runtime, libraries, environment variables, and configuration files.

> 容器通过镜像的拉起而完成加载启动。镜像是一个包含了代码，运行时，调用库，环境变量和配置文件等所有应用必须内容的一个可执行包文件。

A container is a runtime instance of an image--what the image becomes in memory when executed (that is, an image with state, or a user process). You can see a list of your running containers with the command, docker ps, just as you would in Linux.

> 容器是一个镜像运行时的实例，具体就是镜像被执行后加载到内存的内容，也可以理解为一个有了状态的镜像，或是一个用户进程。可以使用docker ps命令来查看当前已运行的容器实例。

### Containers and virtual machines 容器和虚拟机
A container runs natively on Linux and shares the kernel of the host machine with other containers. It runs a discrete process, taking no more memory than any other executable, making it lightweight.

> 容器在Linux上本机运行，并与其他容器共享主机的内核。 它运行一个独立的进程，不占用任何其他可执行文件的内存，使其轻量级。

By contrast, a virtual machine (VM) runs a full-blown “guest” operating system with virtual access to host resources through a hypervisor. In general, VMs provide an environment with more resources than most applications need.

>相比之下，虚拟机（VM）通过管理程序运行一个完整的虚拟化操作系统实现对宿主机资源的访问。 通常，VM提供的是一套全面完整的运行环境，超出了大多数应用程序所需要的范围。

![contrast of docker and vm](	https://img-repo1-1251337292.cos.ap-beijing.myqcloud.com/tech/docker/image/Container-1.png)

### Prepare your Docker environment 准备Docker环境

Install a maintained version of Docker Community Edition (CE) or Enterprise Edition (EE) on a supported platform.

> 在支持Docker的平台上安装长期维护的的Docker版本，包括Community Edition（CE）社区版或Enterprise Edition（EE）企业版。

For full Kubernetes Integration

* Kubernetes on Docker Desktop for Mac is available in 17.12 Edge (mac45) or 17.12 Stable (mac46) and higher.
* Kubernetes on Docker Desktop for Windows is available in 18.02 Edge (win50) and higher edge channels only.

> 对Kubernetes的完全集成：  
> Mac平台的Kubernetes需要Docker 17.12的月度或季度更新版及更高版本  
> Win平台的Kubernetes需要Docker 18.02的月度更新或更高版本

[安装Docker](https://docs.docker.com/install/)

### Test Docker version 查看Docker版本

1. Run docker --version and ensure that you have a supported version of Docker:

> 运行 ```docker --version``` 命令查看是否有支持的Docker版本

```
docker --version

Docker version 17.12.0-ce, build c97c6d6
```

2. Run docker info (or docker version without --) to view even more details about your Docker installation:

> 运行 ```docker info``` 或 ```docker version``` 获取已安装Docker的更多详细信息

```
docker info

Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 17.12.0-ce
Storage Driver: overlay2
...
```

To avoid permission errors (and the use of sudo), add your user to the docker group. Read more.

> 为了避免权限错误（省略sudo命令），请将自己的用户添加至docker用户组（groupadd），[详见...](https://docs.docker.com/install/linux/linux-postinstall/)

### Test Docker installation 检查Docker安装

1. Test that your installation works by running the simple Docker image, hello-world:

> 运行最简单的hello-word镜像检查Docker的安装结果：

```
docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
ca4f61b1923c: Pull complete
Digest: sha256:ca0eeb6fb05351dfc8759c20733c91def84cb8007aa89a5bf606bc8b315b9fc7
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

2. List the hello-world image that was downloaded to your machine:

> 列出在当前服务器上已下载的hello-world镜像：

```
docker image ls
```

3. List the hello-world container (spawned by the image) which exits after displaying its message. If it were still running, you would not need the --all option:

> 列出hello-world镜像拉起时所生成的hello-world容器，容器仅在展示其欢迎信息时属于运行状态，信息显示完毕后需要--all（查看所有状态的容器）查看。如果一个容器处于running运行状态，则不需要额外的--all指令。

```
docker container ls --all

CONTAINER ID     IMAGE           COMMAND      CREATED            STATUS
54f4984ed6a8     hello-world     "/hello"     20 seconds ago     Exited (0) 19 seconds ago
```

### Recap and cheat sheet 命令回顾及总结

```
## List Docker CLI commands
docker
docker container --help

## Display Docker version and info
docker --version
docker version
docker info

## Execute Docker image
docker run hello-world

## List Docker images
docker image ls

## List Docker containers (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq
```

### Conclusion of part one 第一部分总结

Containerization makes CI/CD seamless. For example:

* applications have no system dependencies
* updates can be pushed to any part of a distributed application
* resource density can be optimized.

> 容器化使得CI/CD持续集成能够无缝连接，比如：  
> * 应用不再有系统依赖（所有依赖都打在镜像里）  
> * 在分布式系统应用中可实时更新任一部分的Docker镜像  
> * 主机资源可以被最大化利用

With Docker, scaling your application is a matter of spinning up new executables, not running heavy VM hosts.

> 使用Docker后，扩展应用就是启动新的可执行包，而不再是启动重量级的虚拟机

