## Get Started, Part 3: Services 服务

### Prerequisites 准备

* Install Docker version 1.13 or higher.
* Get Docker Compose. On Docker Desktop for Mac and Docker Desktop for Windows it’s pre-installed, so you’re good-to-go. On Linux systems you need to install it directly. On pre Windows 10 systems without Hyper-V, use Docker Toolbox.
* Read the orientation in Part 1.
* Learn how to create containers in Part 2.
* Make sure you have published the friendlyhello image you created by pushing it to a registry. We use that shared image here.
* Be sure your image works as a deployed container. Run this command, slotting in your info for username, repo, and tag: docker run -p 4000:80 username/repo:tag, then visit http://localhost:4000/.

> * 安装Docker 1.13或更高版本  
> * 获取Docker排版工具。在Mac或Windows平台上是预制安装的，无需额外准备。在Linux系统时你需要直接安装[install](https://github.com/docker/compose/releases)。在Windows 10之前没有```Hyper-V```的系统请使用Docker Toolbox。
> * 阅读第一部分的介绍
> * 学习第二部分关于创建容器的内容
> * 确认你已经打包并发布了```friendlyhello```镜像并推动到了公共仓库，我们在本章会使用到这个公用镜像
> * 确认你的镜像已经被部署为在运行的容器，运行下面的命令，将具体内容替换为你的用户名，镜像名和标签：```docker run -p 4000:80 username/repo:tag```，然后访问```http://localhost:4000/```

### Introduction 介绍

In part 3, we scale our application and enable load-balancing. To do this, we must go one level up in the hierarchy of a distributed application: the service.

* Stack
* Services (you are here)
* Container (covered in part 2)

> 在第3部分，我们扩展自己的应用并且增加负载均衡功能。为了实现这个目标，我们必须上升到分布式应用分层结构的更高一层：服务

> * 服务栈
> * 服务（当前位置）
> * 容器（第2部分内容）

### About services 关于服务

In a distributed application, different pieces of the app are called “services”. For example, if you imagine a video sharing site, it probably includes a service for storing application data in a database, a service for video transcoding in the background after a user uploads something, a service for the front-end, and so on.

> 在分布式应用程序中，应用程序的不同组成部分被称为“服务”。例如，如果您构想一个视频共享网站，它可能包括一个用于在数据库中存储应用数据的服务，一个用于在用户上传内容交给后台进行视频转码的服务，一个用于前端展示的服务，等等。

Services are really just “containers in production.” A service only runs one image, but it codifies the way that image runs—what ports it should use, how many replicas of the container should run so the service has the capacity it needs, and so on. Scaling a service changes the number of container instances running that piece of software, assigning more computing resources to the service in the process.

> 服务实际上只是“生产环境中的容器”。一个服务只运行一个镜像，但它编写了镜像的运行方式，包括它应该使用哪些端口，运行多少个容器副本，以满足服务所需的吞吐量等等。扩展服务会更改运行该程序的容器实例的数量，从而为流程中的服务分配更多的计算资源。

Luckily it’s very easy to define, run, and scale services with the Docker platform -- just write a docker-compose.yml file.

> 幸运的是，在Docker平台上，定义，运行并扩展服务的过程非常简单，仅需要编写一个docker-compose.yml编排文件。

### Your first docker-compose.yml file 第一个编排文件

A docker-compose.yml file is a YAML file that defines how Docker containers should behave in production.

> 一个docker-compose.yml编排文件是基于YAML格式的文件，其定义了Docker容器怎样在生产环境中运行。

#### docker-compose.yml

Save this file as docker-compose.yml wherever you want. Be sure you have pushed the image you created in Part 2 to a registry, and update this .yml by replacing username/repo:tag with your image details.

> 在任意位置保存下面的文件为docker-compose.yml。确认你已经发布了第2部分所创建的镜像至公共仓库并且将下面文件中的username/repo:tag替换为你实际镜像仓库中的值。

```
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "4000:80"
    networks:
      - webnet
networks:
  webnet:
```

This docker-compose.yml file tells Docker to do the following:

* Pull the image we uploaded in step 2 from the registry.
* Run 5 instances of that image as a service called web, limiting each one to use, at most, 10% of a single core of CPU time (this could also be e.g. “1.5” to mean 1 and half core for each), and 50MB of RAM.
* Immediately restart containers if one fails.
* Map port 4000 on the host to web’s port 80.
* Instruct web’s containers to share port 80 via a load-balanced network called webnet. (Internally, the containers themselves publish to web’s port 80 at an ephemeral port.)
* Define the webnet network with the default settings (which is a load-balanced overlay network).

> 上面的编排文件通知Docker做了下面的操作：

> * 从远程仓库拉取第2部分所上传的镜像
> * 在名为web的服务下运行5个镜像实例，对每个实例作资源使用的限制，每个实例使用最多不超过10%的cpu和50MB的内存（cpu为1.5即不超过150%）
> * 当有容器运行失败时，立刻重启容器
> * 将服务器4000端口映射到web服务的80端口
> * 通知web服务的容器实例通过一个名为webnet的负载均衡网络共享80端口（在内部，容器实例使用临时端口去对接web服务的80端口）
> * 使用默认配置定义webnet网络（负载均衡的覆盖网络）

### Run your new load-balanced app 运行新的负载均衡应用

Before we can use the ```docker stack deploy``` command we first run:

> 在我们使用```docker stack deploy``命令之前，先运行：

```
docker swarm init
```

Note: We get into the meaning of that command in part 4. If you don’t run docker swarm init you get an error that “this node is not a swarm manager.”

> 注意：我们将会在第4部分详细介绍上面命令的含义。如果不运行docker swarm init命令，你将看到```this node is not a swarm manager.```的错误返回。

Now let’s run it. You need to give your app a name. Here, it is set to getstartedlab:

> 现在让我们运行一遍。你需要为你的应用起一个名字。在这里我们设置成```getstartedlab```:

```
docker stack deploy -c docker-compose.yml getstartedlab
```

Our single service stack is running 5 container instances of our deployed image on one host. Let’s investigate.

Get the service ID for the one service in our application:

> 我们的独立服务栈运行在了基于一个镜像的5个容器实例上，并且是在同一个服务器上。让我们研究一下。

> 获取我们应用中唯一服务的服务ID：

```
docker service ls
显示getstartedlab_web
docker stack ls
显示getstartedlab
```

Look for output for the web service, prepended with your app name. If you named it the same as shown in this example, the name is getstartedlab_web. The service ID is listed as well, along with the number of replicas, image name, and exposed ports.

> 以你应用的名称为前缀，查找Web服务的输出。如果您将其命名为与此示例中显示的相同，则名称为getstartedlab_web。还列出了服务ID，以及副本数，镜像名称和公开端口

```
docker stack services getstartedlab
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
bqpve1djnk0x        getstartedlab_web   replicated          5/5                 username/repo:tag   *:4000->80/tcp
```

A single container running in a service is called a task. Tasks are given unique IDs that numerically increment, up to the number of replicas you defined in docker-compose.yml. List the tasks for your service:

> 在服务中运行的单个容器成为任务，任务会被赋予以数字递增的直到编排文件中副本数上限的唯一ID。列出服务中的任务：

```
docker service ps getstartedlab_web
```

Tasks also show up if you just list all the containers on your system, though that is not filtered by service:

> 在使用命令列出所有容器时，即使没有按服务过滤，所有的任务也会被显示出来

```
docker container ls -q

--help
List containers
Aliases:
  ls, ps, list
Options:
  -a, --all             Show all containers (default shows just running)
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print containers using a Go template
  -n, --last int        Show n last created containers (includes all states) (default -1)
  -l, --latest          Show the latest created container (includes all states)
      --no-trunc        Don't truncate output
  -q, --quiet           Only display numeric IDs
  -s, --size            Display total file sizes
```

You can run ```curl -4 http://localhost:4000``` several times in a row, or go to that URL in your browser and hit refresh a few times.

> 用户可以执行多次```curl -4 http://localhost:4000```或在浏览器中多次刷新页面

Either way, the container ID changes, demonstrating the load-balancing; with each request, one of the 5 tasks is chosen, in a round-robin fashion, to respond. The container IDs match your output from the previous command 

> 页面中容器ID的变化证明了负载均衡器的存在，每次请求会被转发到5个后端任务中的一个并返回结果，默认是round-robi的轮训方式。容器ID与上面命令docker container ls列出的ID是一致的。

To view all tasks of a stack, you can run docker stack ps followed by your app name, as shown in the following example:

> 如果需要列出服务栈的所有任务，可以使用```docker stack ps```并加上应用的名称，如下所示：

```
docker stack ps getstartedlab
ID                  NAME                  IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
uwiaw67sc0eh        getstartedlab_web.1   username/repo:tag   docker-desktop      Running             Running 9 minutes ago                       
sk50xbhmcae7        getstartedlab_web.2   username/repo:tag   docker-desktop      Running             Running 9 minutes ago                       
c4uuw5i6h02j        getstartedlab_web.3   username/repo:tag   docker-desktop      Running             Running 9 minutes ago                       
0dyb70ixu25s        getstartedlab_web.4   username/repo:tag   docker-desktop      Running             Running 9 minutes ago                       
aocrb88ap8b0        getstartedlab_web.5   username/repo:tag   docker-desktop      Running             Running 9 minutes ago
```

* Running Windows 10?  
Windows 10 PowerShell should already have curl available, but if not you can grab a Linux terminal emulator like Git BASH, or download wget for Windows which is very similar.

> * Windows 10中运行？  
> Windows 10中的终端里应该已包含了curl命令，不过也可以下载Linux的终端模拟器如Git Bash或者下载Windows下小巧的的wget命令。

* Slow response times?  
Depending on your environment’s networking configuration, it may take up to 30 seconds for the containers to respond to HTTP requests. This is not indicative of Docker or swarm performance, but rather an unmet Redis dependency that we address later in the tutorial. For now, the visitor counter isn’t working for the same reason; we haven’t yet added a service to persist data.

> * 响应缓慢？  
> 根据不同用户环境的网络配置，容器最多可能需要30秒才能返回响应的HTTP请求。这并不能说明Docker或swarm性能，而是服务没有满足Redis的依赖性，这点我们后面会详细讨论。当前访客计数功能由于同样的原因不能服务，那就是我们还没有添加服务来保存数据。

### Scale the app 扩展应用

You can scale the app by changing the replicas value in docker-compose.yml, saving the change, and re-running the docker stack deploy command:

> 用户可以通过更改编排文件中的服务数量来实现扩展应用的目的，保存更新后的编排文件后重新运行```docker stack deploy```命令即可：

```
docker stack deploy -c docker-compose.yml getstartedlab
```

Docker performs an in-place update, no need to tear the stack down first or kill any containers.

Now, re-run docker container ls -q to see the deployed instances reconfigured. If you scaled up the replicas, more tasks, and hence, more containers, are started.

> Docker使用就地更新策略，不需要提前停止stack，或杀掉容器进程。  
> 现在重新执行```docker container ls -q```可以看到已部署的容器实例已被重新配置。如果用户扩展了服务署，或更多任务，那么将会有更多的容器被启动运行。

#### Take down the app and the swarm 停止应用和swarm

* Take the app down with docker stack rm:

> 使用```docker stack rm```停止应用

```
docker stack rm getstartedlab
```

* 停止swarm

```
docker swarm leave --force
```

It’s as easy as that to stand up and scale your app with Docker. You’ve taken a huge step towards learning how to run containers in production. Up next, you learn how to run this app as a bonafide swarm on a cluster of Docker machines.

> 使用Docker启动和扩展应用是非常简单的。到目前为止已经向如何在生产环境中运行容器迈进了一大步。接下来，将学习在容器集群中如何把应用发布在真正的swarm管理集群中。

Note: Compose files like this are used to define applications with Docker, and can be uploaded to cloud providers using Docker Cloud, or on any hardware or cloud provider you choose with Docker Enterprise Edition.

> 注意：像上面这样的编排文件方式适用于定义Docker应用，并且可以上传到使用了Docker云或其他任何用户选择使用的企业版Docker所支持的硬件、云提供方来提供使用。

### Recap and cheat sheet (optional) 回顾与小结

To recap, while typing docker run is simple enough, the true implementation of a container in production is running it as a service. Services codify a container’s behavior in a Compose file, and this file can be used to scale, limit, and redeploy our app. Changes to the service can be applied in place, as it runs, using the same command that launched the service: docker stack deploy.

> 回顾上文，虽然使用docker run命令非常简单，实际在生产中，容器的实现是被放在服务中定义并运行的。服务在Compose编排文件中定义了容器的行为，编排文件可用于扩展，限制和重新部署应用程序。服务的更新是可以在运行过程中被就地执行的，与第一次启动服务时的命令相同，使用```docker stack deploy```即可重新生效。

Some commands to explore at this stage:

> 本部分包含的一些值得回顾和探索的命令如下：

```
docker stack ls                                            # List stacks or apps
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
docker service ls                 # List running services associated with an app
docker service ps <service>                  # List tasks associated with an app
docker inspect <task or container>                   # Inspect task or container
docker container ls -q                                      # List container IDs
docker stack rm <appname>                             # Tear down an application
docker swarm leave --force      # Take down a single node swarm from the manager
docker stats
```


