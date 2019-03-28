## Orientation and setup 介绍与安装

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


