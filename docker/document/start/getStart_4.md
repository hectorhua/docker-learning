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

#### Configure a docker-machine shell to the swarm manager 为swarm集群管理节点配置docker-machine的shell环境

So far, you’ve been wrapping Docker commands in docker-machine ssh to talk to the VMs. Another option is to run docker-machine env <machine> to get and run a command that configures your current shell to talk to the Docker daemon on the VM. This method works better for the next step because it allows you to use your local docker-compose.yml file to deploy the app “remotely” without having to copy it anywhere.

> 到目前为止，用户使用```docker-machine ssh```将命令发送到虚拟机。另外一种可行的方案是配置用户当前的shell环境使得能够直接向虚拟机上运行的Docker后端发送命令。这种方式在后面的操作步骤中更加便捷，因为可以将用户本地的编排文件直接部署到远程的节点，而不需要先把文件拷贝过去。

Type docker-machine env myvm1, then copy-paste and run the command provided as the last line of the output to configure your shell to talk to myvm1, the swarm manager.

> 发送```docker-machine env myvm1```命令，然后将返回结果的最后一行复制并运行，来实现配置本地终端连接到管理节点myvm1。

The commands to configure your shell differ depending on whether you are Mac, Linux, or Windows, so examples of each are shown on the tabs below.

> 在不同平台下，上面的配置命令有所差别，下面以Mac和Linux为例展示：

DOCKER MACHINE SHELL ENVIRONMENT ON MAC OR LINUX

Run docker-machine env myvm1 to get the command to configure your shell to talk to myvm1.

> 执行```docker-machine env myvm1 ```配置连接myvm1：

```
$ docker-machine env myvm1
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/sam/.docker/machine/machines/myvm1"
export DOCKER_MACHINE_NAME="myvm1"
# Run this command to configure your shell:
# eval $(docker-machine env myvm1)
```

Run the given command to configure your shell to talk to myvm1.

> 执行给出的命令

```
eval $(docker-machine env myvm1)
```

Run docker-machine ls to verify that myvm1 is now the active machine, as indicated by the asterisk next to it.

> 执行```docker-machine ls```确认myvm1现在是一个有效状态的服务器，查看下面显示的星号*是否存在：

```
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
myvm1   *        virtualbox   Running   tcp://192.168.99.100:2376           v17.06.2-ce
myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.06.2-ce
```

#### Deploy the app on the swarm manager 通过swarm集群管理节点部署应用

Now that you have myvm1, you can use its powers as a swarm manager to deploy your app by using the same docker stack deploy command you used in part 3 to myvm1, and your local copy of docker-compose.yml.. This command may take a few seconds to complete and the deployment takes some time to be available. Use the docker service ps <service_name> command on a swarm manager to verify that all services have been redeployed.

> 现在用户已经拥有了myvm1节点，可以使用这个管理节点来按照第3部分的流程部署同样的服务栈来发布应用，使用```docker stack deploy```命令和前面的docker-compose.yml编排文件部署应用到myvm1节点。这个命令需要花费几秒的时间完成部署并在一段时间后提供可用的服务。在swarm管理节点上可以使用```docker service ps <service_name>```命令确认所有的服务被重新部署。

You are connected to myvm1 by means of the docker-machine shell configuration, and you still have access to the files on your local host. Make sure you are in the same directory as before, which includes the docker-compose.yml file you created in part 3.

> 用户现在通过docker-machine终端配置来连接到myvm1节点，并且还是在自己本地文件的访问权限下。确保当前处于之前创建的目录下，应当包含第3部分的```docker-compose.yml```文件。

Just like before, run the following command to deploy the app on myvm1.

> 类似之前的部署操作，执行：

```
docker stack deploy -c docker-compose.yml getstartedlab
```

And that’s it, the app is deployed on a swarm cluster!

> 就是这样，应用被部署到了swarm集群上啦！  

***Note:*** If your image is stored on a private registry instead of Docker Hub, you need to be logged in using docker login <your-registry> and then you need to add the --with-registry-auth flag to the above command. For example:

> 注意：如果用户的镜像存储在了私有仓库上而不是公共的Docker Hub，用户需要首先```docker login <your-registry>```登录仓库，然后在部署命令中添加```--with-registry-auth```标志，如：

```
docker login registry.example.com
docker stack deploy --with-registry-auth -c docker-compose.yml getstartedlab
```

This passes the login token from your local client to the swarm nodes where the service is deployed, using the encrypted WAL logs. With this information, the nodes are able to log into the registry and pull the image.

> 这个步骤会将本地登陆仓库后的token传递到swarm节点，应用在对应节点部署时会使用token拉取镜像，token传递方式为加密后的WAL logs。

Now you can use the same docker commands you used in part 3. Only this time notice that the services (and associated containers) have been distributed between both myvm1 and myvm2.

> 现在用户可使用第3部分中用过的命令来部署应用，这次仅需要注意服务（相关容器）会分布到myvm1和myvm2两个节点上。

```
$ docker stack ps getstartedlab

ID            NAME                  IMAGE                   NODE   DESIRED STATE
jq2g3qp8nzwx  getstartedlab_web.1   gordon/get-started:part2  myvm1  Running
88wgshobzoxl  getstartedlab_web.2   gordon/get-started:part2  myvm2  Running
vbb1qbkb0o2z  getstartedlab_web.3   gordon/get-started:part2  myvm2  Running
ghii74p9budx  getstartedlab_web.4   gordon/get-started:part2  myvm1  Running
0prmarhavs87  getstartedlab_web.5   gordon/get-started:part2  myvm2  Running
```

Connecting to VMs with docker-machine env and docker-machine ssh

> 使用```docker-machine env```和```docker-machine ssh```连接到服务节点

* To set your shell to talk to a different machine like myvm2, simply re-run docker-machine env in the same or a different shell, then run the given command to point to myvm2. This is always specific to the current shell. If you change to an unconfigured shell or open a new one, you need to re-run the commands. Use docker-machine ls to list machines, see what state they are in, get IP addresses, and find out which one, if any, you are connected to. To learn more, see the Docker Machine getting started topics.
* Alternatively, you can wrap Docker commands in the form of docker-machine ssh <machine> "<command>", which logs directly into the VM but doesn’t give you immediate access to files on your local host.
* On Mac and Linux, you can use docker-machine scp <file> <machine>:~ to copy files across machines, but Windows users need a Linux terminal emulator like Git Bash for this to work.

> * 如果想发送命令到另一个不同的服务节点myvm2，只需重新运行```docker-machine env```命令，在当前shell或新开启的shell中都可以，然后按照给出的提示加入到myvm2。这种方式决定了当前的shell只能连接一个节点，如果每次想变更连接或重新打开新的shell，就必须重新执行以上的命令。使用```docker-machine ls```可以列出当前的服务器节点，查看它们的状态，分配的IP，选择需要连接的节点。更多内容见[Docker Machine getting started topics.](https://docs.docker.com/machine/get-started/#create-a-machine)
> * 另一种可选的方式，可以将Docker命令包裹在```docker-machine ssh <machine> "<command>"```之中，这种方式类似登陆到节点上执行了命令，但是不在拥有本地文件的访问权限了。
> * 在Mac或Linux系统中，用户可以使用```docker-machine scp <file> <machine>:~```将本地文件拷贝到服务器节点上（弥补了上面第2中方式的缺陷），但是Windows用户需要Linux的终端模拟器（如GitBash）才能实现。

This tutorial demos both docker-machine ssh and docker-machine env, since these are available on all platforms via the docker-machine CLI.

> ```docker-machine ssh```和```docker-machine env```都是基于```docker-machine```命令接口实现的，并且在任意平台上都能运行。

#### Accessing your cluster 访问集群

You can access your app from the IP address of either myvm1 or myvm2.

> 用户可以通过myvm1或myvm2的IP地址来访问应用

The network you created is shared between them and load-balancing. Run docker-machine ls to get your VMs’ IP addresses and visit either of them on a browser, hitting refresh (or just curl them).

> 用户创建的网络会在用户和负载均衡之间共享。使用```docker-machine ls```查看服务器节点的IP地址，并在浏览器中访问（或curl）。

There are five possible container IDs all cycling by randomly, demonstrating the load-balancing.

> 目前有5个可用的容器在被随机循环的请求，证明了负载均衡器的有效可用。

The reason both IP addresses work is that nodes in a swarm participate in an ingress routing mesh. This ensures that a service deployed at a certain port within your swarm always has that port reserved to itself, no matter what node is actually running the container. Here’s a diagram of how a routing mesh for a service called my-web published at port 8080 on a three-node swarm would look:

> 那么，服务器节点的IP地址能够被访问的原因是，服务器节点在swarm集群中扮演了一个```ingress routing mesh```路由网络入口的角色。这点保证了被部署在swarm集群中某个端口（编排文件中的容器端口:节点端口）的服务，永远能够为自己保留其端口（节点端口），不管服务的容器具体运行在哪个节点上，都可以通过节点端口访问到服务应用。下面是关于路由网络如何将my-web服务发布到拥有三个服务器节点的集群的8080端口的图解：

![](https://img-repo1-1251337292.cos.ap-beijing.myqcloud.com/tech/docker/image/Container-2.png)


***Having connectivity trouble?***

Keep in mind that to use the ingress network in the swarm, you need to have the following ports open between the swarm nodes before you enable swarm mode:

* Port 7946 TCP/UDP for container network discovery.
* Port 4789 UDP for the container ingress network.

Double check what you have in the ports section under your web service and make sure the ip addresses you enter in your browser or curl reflects that

> 记住swarm集群中的网络准入模式，在启用swarm集群之前，用户需要打开节点的下列端口：

> * 端口7946 TCP/UDP 用于容器网络互相发现
> * 端口4789 UDP 用于容器网络准入

> 再次检查应用服务关于端口部分的配置，并确保输入浏览器的IP地址和端口能体现出配置的有效性

### Iterating and scaling your app 迭代并扩展应用

From here you can do everything you learned about in parts 2 and 3.

> 从现在开始用户可以根据第2、3部分所学的内容，任意操作容器了

Scale the app by changing the docker-compose.yml file.

> 通过docker-compose.yml编排文件来扩展应用

Change the app behavior by editing code, then rebuild, and push the new image. (To do this, follow the same steps you took earlier to build the app and publish the image).

> 通过编辑代码，重新构建推送镜像来更新应用服务（按照之前步骤构建应用并推送发布镜像）。

In either case, simply run docker stack deploy again to deploy these changes.

> 在之前的例子中，只需要重新运行```docker stack deploy```命令就可以完成应用的更新了

You can join any machine, physical or virtual, to this swarm, using the same docker swarm join command you used on myvm2, and capacity is added to your cluster. Just run docker stack deploy afterwards, and your app can take advantage of the new resources.

> 用户可以将无论是物理机还是虚拟机的服务器加入到swarm集群，如上面使用的命令```docker swarm join```将myvm2加入集群，这样myvm2的计算能力就加入到集群了。接下来只需使用```docker stack deploy```便能向新的计算节点部署容器。

### Cleanup and reboot 清理和重启

#### Stacks and swarms 服务栈和集群

You can tear down the stack with docker stack rm. For example:

> 使用```docker stack rm```命令结束stack，如：

```
docker stack rm getstartedlab
```

***Keep the swarm or remove it?***  
At some point later, you can remove this swarm if you want to with docker-machine ssh myvm2 "docker swarm leave" on the worker and docker-machine ssh myvm1 "docker swarm leave --force" on the manager, but you need this swarm for part 5, so keep it around for now.

> ***保留或者删除swarm？***  
> 某些时刻可以删除swarm集群，使用```docker-machine ssh myvm2 "docker swarm leave"```移除工作节点myvm2，使用```docker-machine ssh myvm1 "docker swarm leave --force"```移除管理节点。但是在第5部分我们还需要用到这个swarm集群，尽量还是先保留吧～

#### Unsetting docker-machine shell variable settings 取消docker-machine shell的变量配置

You can unset the docker-machine environment variables in your current shell with the given command.

On Mac or Linux the command is:

> 用户可以取消docker-machine环境变量的设置，当然仅仅对当前shell有效，命令如下：  
> Mac或Linux命令：

```
eval $(docker-machine env -u)
```

> Windows命令：

```
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env -u | Invoke-Expression
```

This disconnects the shell from docker-machine created virtual machines, and allows you to continue working in the same shell, now using native docker commands (for example, on Docker Desktop for Mac or Docker Desktop for Windows). To learn more, see the Machine topic on unsetting environment variables.

> 上面操作会中断用户与docker-machine创建出来的服务节点之间的shell通信，所有shell都恢复成统一的本地shell，现在只能使用原生的docker命令。了解更多[Machine topic on unsetting environment variables.](https://docs.docker.com/machine/get-started/#unset-environment-variables-in-the-current-shell)

#### Restarting Docker machines 重启Docker节点

If you shut down your local host, Docker machines stops running. You can check the status of machines by running docker-machine ls.

> 如果用户关闭了本地服务器，Docker节点将停止运行。可使用```docker-machine ls```查看docker服务器节点状态：

```
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL   SWARM   DOCKER    ERRORS
myvm1   -        virtualbox   Stopped                 Unknown
myvm2   -        virtualbox   Stopped                 Unknown
```

To restart a machine that’s stopped, run:

> 重启已被关闭的docker节点：

```
docker-machine start <machine-name>
```

如：

```
$ docker-machine start myvm1
Starting "myvm1"...
(myvm1) Check network to re-create if needed...
(myvm1) Waiting for an IP...
Machine "myvm1" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.

$ docker-machine start myvm2
Starting "myvm2"...
(myvm2) Check network to re-create if needed...
(myvm2) Waiting for an IP...
Machine "myvm2" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.
```

### Recap and cheat sheet (optional) 回顾与小节

In part 4 you learned what a swarm is, how nodes in swarms can be managers or workers, created a swarm, and deployed an application on it. You saw that the core Docker commands didn’t change from part 3, they just had to be targeted to run on a swarm master. You also saw the power of Docker’s networking in action, which kept load-balancing requests across containers, even though they were running on different machines. Finally, you learned how to iterate and scale your app on a cluster.

> 在第4部分，用户了解了swarm集群，集群中的节点可以成为管理节点或工作节点，创建swarm以及向集群部署应用。Docker核心的命令与第3部分的内容一样，仅仅是运行的目标变成了swarm集群。在此体验了Docker网络分发的能力，无论容器运行在哪个节点上都能准确的收到负载均衡器转发的请求。最后，用户了解了在集群中迭代和扩展应用的功能。

Here are some commands you might like to run to interact with your swarm and your VMs a bit:

> 以下列出了可能会经常使用到的在swarm集群与服务器节点间交互的命令：

```
docker-machine create --driver virtualbox myvm1 # Create a VM (Mac, Win7, Linux)
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm1 # Win10
docker-machine env myvm1                # View basic information about your node
docker-machine ssh myvm1 "docker node ls"         # List the nodes in your swarm
docker-machine ssh myvm1 "docker node inspect <node ID>"        # Inspect a node
docker-machine ssh myvm1 "docker swarm join-token -q worker"   # View join token
docker-machine ssh myvm1   # Open an SSH session with the VM; type "exit" to end
docker node ls                # View nodes in swarm (while logged on to manager)
docker-machine ssh myvm2 "docker swarm leave"  # Make the worker leave the swarm
docker-machine ssh myvm1 "docker swarm leave -f" # Make master leave, kill swarm
docker-machine ls # list VMs, asterisk shows which VM this shell is talking to
docker-machine start myvm1            # Start a VM that is currently not running
docker-machine env myvm1      # show environment variables and command for myvm1
eval $(docker-machine env myvm1)         # Mac command to connect shell to myvm1
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression   # Windows command to connect shell to myvm1
docker stack deploy -c <file> <app>  # Deploy an app; command shell must be set to talk to manager (myvm1), uses local Compose file
docker-machine scp docker-compose.yml myvm1:~ # Copy file to node's home dir (only required if you use ssh to connect to manager and deploy the app)
docker-machine ssh myvm1 "docker stack deploy -c <file> <app>"   # Deploy an app using ssh (you must have first copied the Compose file to myvm1)
eval $(docker-machine env -u)     # Disconnect shell from VMs, use native docker
docker-machine stop $(docker-machine ls -q)               # Stop all running VMs
docker-machine rm $(docker-machine ls -q) # Delete all VMs and their disk images
```


