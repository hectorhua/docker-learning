## Get Started, Part 4: Swarms 集群

### Prerequisites 准备

* Install Docker version 1.13 or higher.
* Get Docker Compose as described in Part 3 prerequisites.
* Get Docker Machine, which is pre-installed with Docker Desktop for Mac and Docker Desktop for Windows, but on Linux systems you need to install it directly. On pre Windows 10 systems without Hyper-V, as well as Windows 10 Home, use Docker Toolbox.
* Read the orientation in Part 1.
* Learn how to create containers in Part 2.
* Make sure you have published the friendlyhello image you created by pushing it to a registry. We use that shared image here.
* Be sure your image works as a deployed container. Run this command, slotting in your info for username, repo, and tag: docker run -p 80:80 username/repo:tag, then visit http://localhost/.
* Have a copy of your docker-compose.yml from Part 3 handy.

> * 安装Docker 1.13或更高版本
> * 准备Docker编排内容，参见第3部分内容
> * 准备Docker服务器，在Mac或Windows平台上是预制安装的，无需额外准备。在Linux系统时你需要直接安装install。在Windows 10之前没有Hyper-V的系统请使用Docker Toolbox
> * 阅读第一部分的介绍
> * 学习第二部分关于创建容器的内容
> * 确认你已经打包并发布了friendlyhello镜像并推动到了公共仓库，我们在本章会使用到这个公用镜像
> * 确认你的镜像已经被部署为在运行的容器，运行下面的命令，将具体内容替换为你的用户名，镜像名和标签：docker run -p 4000:80 username/repo:tag，然后访问http://localhost:4000/
> * 从第3部分手册中拷贝出一份docker-compose.yml编排文件

### Introduction

In part 3, you took an app you wrote in part 2, and defined how it should run in production by turning it into a service, scaling it up 5x in the process.

> 在第3部分，用户使用了第2部分编写的应用，并通过注入到服务方式定义了其运行行为，还对其进行了水平扩展至5个实例。

Here in part 4, you deploy this application onto a cluster, running it on multiple machines. Multi-container, multi-machine applications are made possible by joining multiple machines into a “Dockerized” cluster called a swarm.

> 在第4部分，用户将应用部署至集群，并在多个服务器上运行。多容器，多服务器的应用是由被称作swarm的集群管理工具将其注入容器化的集群中。

### Understanding Swarm clusters 理解Swarm集群

A swarm is a group of machines that are running Docker and joined into a cluster. After that has happened, you continue to run the Docker commands you’re used to, but now they are executed on a cluster by a swarm manager. The machines in a swarm can be physical or virtual. After joining a swarm, they are referred to as nodes.

> 一个Swarm集群就是运行了Docker并且组成为集群的一组服务器。当Swarm集群建立之后，用户可以继续使用平时的Docker命令，但是它们是在由swarm管理的集群上被执行。Swarm集群中的服务器可以是物理机或者是虚拟机，当加入到集群后，它们被称作节点。

Swarm managers can use several strategies to run containers, such as “emptiest node” -- which fills the least utilized machines with containers. Or “global”, which ensures that each machine gets exactly one instance of the specified container. You instruct the swarm manager to use these strategies in the Compose file, just like the one you have already been using.

> Swarm集群管理可以配置多种策略来调度容器，比如：  
> emptiest node最空闲：把所有容器调度到最少的节点上  
> global最全面：保证每个节点都调度到1个指定的容器实例  
> 用户可以在编排文件中配置指定swarm使用哪种策略，就像之前使用配置的方式。

Swarm managers are the only machines in a swarm that can execute your commands, or authorize other machines to join the swarm as workers. Workers are just there to provide capacity and do not have the authority to tell any other machine what it can and cannot do.

> Swarm集群管理是集群中唯一能够执行命令的服务器，或者授权其他服务器作为工作节点加入集群。工作节点仅提供计算能力完成任务，而没有权限去指定其他节点的工作任务。

Up until now, you have been using Docker in a single-host mode on your local machine. But Docker also can be switched into swarm mode, and that’s what enables the use of swarms. Enabling swarm mode instantly makes the current machine a swarm manager. From then on, Docker runs the commands you execute on the swarm you’re managing, rather than just on the current machine.

> 到目前为止，用户仅仅在一个本地的单台服务器上运行了Docker。但是Docker也可以选择切换到swarm模式，这就是启用swarm的原因。开启swarm模式后会使当前的服务器称为集群管理者。从这个时刻开始，Docker运行的命令就是在swarm上执行的，而不再是你当前的服务器上。

### Set up your swarm 创建swarm集群

A swarm is made up of multiple nodes, which can be either physical or virtual machines. The basic concept is simple enough: run docker swarm init to enable swarm mode and make your current machine a swarm manager, then run docker swarm join on other machines to have them join the swarm as workers. Choose a tab below to see how this plays out in various contexts. We use VMs to quickly create a two-machine cluster and turn it into a swarm.

> 一个swarm集群是由多个节点组成，无论是物理机还是虚拟机。基本的概念很简单：运行```docker swarm init```启用swarm模式，使当前机器称为管理者，然后在其他节点上运行```docker swarm join```来以工作节点角色加入这个集群。选择下面的标签，查看在各种情况下的结果。我们使用VM虚拟机快速创建一个双机群集并将其转换为swarm群集。

#### Create a cluster 创建集群

VMS ON YOUR LOCAL MACHINE (MAC, LINUX, WINDOWS 7 AND 8)

You need a hypervisor that can create virtual machines (VMs), so install Oracle VirtualBox for your machine’s OS.

> 用户需要hypervisor管理工具来创建虚拟机，可以选择VirtualBox来作为服务器的操作系统。

Note: If you are on a Windows system that has Hyper-V installed, such as Windows 10, there is no need to install VirtualBox and you should use Hyper-V instead. View the instructions for Hyper-V systems by clicking the Hyper-V tab above. If you are using Docker Toolbox, you should already have VirtualBox installed as part of it, so you are good to go.

> 注意：在Windows系统上，如果Hyper-V已被安装，比如Windows 10，那么就需不要安装VirtualBox，而是用已有的Hyper-V代替。具体参见Windows下的创建集群说明。

Now, create a couple of VMs using docker-machine, using the VirtualBox driver:

> 现在创建两个虚拟机，使用```docker-machine```命令加VirtualBox驱动：

```
docker-machine create --driver virtualbox myvm1
docker-machine create --driver virtualbox myvm2
```

#### LIST THE VMS AND GET THEIR IP ADDRESSES 列出虚拟机并获取其IP

You now have two VMs created, named myvm1 and myvm2.

> 现在虚拟机创建完成，名为myvm1和myvm2

Use this command to list the machines and get their IP addresses.

> 使用下面命令列出服务器和其IP地址：

```
docker-machine ls
```

Here is example output from this command.

> 示例

```
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
myvm1   -        virtualbox   Running   tcp://192.168.99.100:2376           v17.06.2-ce
myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.06.2-ce
```

#### INITIALIZE THE SWARM AND ADD NODES 初始化Swarm和节点

The first machine acts as the manager, which executes management commands and authenticates workers to join the swarm, and the second is a worker.

> 将第一个服务节点作为管理者，负责执行管理命令和授权其他工作节点加入集群，第二个服务节点作为工作节点。

You can send commands to your VMs using docker-machine ssh. Instruct myvm1 to become a swarm manager with docker swarm init and look for output like this:

> 用户可以使用```docker-machine ssh```命令向其他虚拟机节点发送指令。使用```docker swarm init```指定myvm1成为swarm管理者，查看输出内容：

```
$ docker-machine ssh myvm1 "docker swarm init --advertise-addr <myvm1 ip>"
Swarm initialized: current node <node ID> is now a manager.

To add a worker to this swarm, run the following command:

  docker swarm join \
  --token <token> \
  <myvm ip>:<port>

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

* Ports 2377 and 2376  
Always run docker swarm init and docker swarm join with port 2377 (the swarm management port), or no port at all and let it take the default.  
The machine IP addresses returned by docker-machine ls include port 2376, which is the Docker daemon port. Do not use this port or you may experience errors.

> * 端口2377和2376  
> 始终使用2377端口（swarm管理端口）运行```docker swarm init```和```docker swarm join```，或者不指定端口而使用默认端口  

* Having trouble using SSH? Try the --native-ssh flag   
Docker Machine has the option to let you use your own system’s SSH, if for some reason you’re having trouble sending commands to your Swarm manager. Just specify the --native-ssh flag when invoking the ssh command:

> * 无法使用SSH？试用下--native-ssh参数  
> Docker服务器可通过参数选择使用用户自身系统的SSH登陆服务，以防特殊原因导致用户无法向Swarm管理者发送命令。只需在调用命令时指定参数```--native-ssh```

```
docker-machine --native-ssh ssh myvm1 ...
```

As you can see, the response to docker swarm init contains a pre-configured docker swarm join command for you to run on any nodes you want to add. Copy this command, and send it to myvm2 via docker-machine ssh to have myvm2 join your new swarm as a worker:

> 如上所见，```docker swarm init```返回的结果中包含预置的```docker swarm join```命令，以便用户在任何想要加入的节点上自动运行。将join命令发送至myvm2使第2个节点作为工作节点加入到集群。

```
$ docker-machine ssh myvm2 "docker swarm join \
--token <token> \
<ip>:2377"

This node joined a swarm as a worker.
```

Congratulations, you have created your first swarm!

> 恭喜，你成功创建了第一个swarm集群！

Run docker node ls on the manager to view the nodes in this swarm:

> 在集群的管理节点执行```docker node ls```来查看swarm中的所有节点：
```
$ docker-machine ssh myvm1 "docker node ls"
ID                            HOSTNAME            STATUS           
AVAILABILITY        MANAGER STATUS
brtu9urxwfd5j0zrmkubhpkbd     myvm2               Ready               Active
rihwohkh3ph38fhillhhb84sk *   myvm1               Ready               Active              Leader
```

* Leaving a swarm  
If you want to start over, you can run docker swarm leave from each node.

> * 移除swarm集群  
如果想要重新添加节点，可以在节点上使用```docker swarm leave```

### Deploy your app on the swarm cluster 在swarm集群中部署应用

The hard part is over. Now you just repeat the process you used in part 3 to deploy on your new swarm. Just remember that only swarm managers like myvm1 execute Docker commands; workers are just for capacity.

> 最难的部分已经过去啦。现在仅需要重复第3部分的内容来向swarm集群中部署应用。需要牢记的是，只有管理者节点myvm1才能执行Docker指令，其他工作节点仅提供计算。





