## Get Started, Part 2: Containers 容器

### Prerequisites 准备

* Install Docker version 1.13 or higher.
* Read the orientation in Part 1.
* Give your environment a quick test run to make sure you’re all set up:

```
docker run hello-world
```

> * 安装Docker 1.13或更高版本  
> * 阅读第一部分的Docker介绍  
> * 使用下面命令快速检查当前Docker环境是否全部安装完毕：

```
docker run hello-world
```

### Introduction 介绍

It’s time to begin building an app the Docker way. We start at the bottom of the hierarchy of such app, a container, which this page covers. Above this level is a service, which defines how containers behave in production, covered in Part 3. Finally, at the top level is the stack, defining the interactions of all the services, covered in Part 5.

> 现在开始使用Docker方式构建应用程序。我们从这个应用程序的底层构建开始，即容器，也是本页面主要的内容。在此层次之上的是服务，服务定义了容器在生产环境中的交互行为，具体将在第3部分详细介绍。最后，在顶层的是服务栈，定义了第5部分中介绍的有关服务的交互。

* Stack
* Services
* Container (you are here)

> * 服务栈
> * 服务
> * 容器（当前位置）

### Your new development environment 新的开发环境

In the past, if you were to start writing a Python app, your first order of business was to install a Python runtime onto your machine. But, that creates a situation where the environment on your machine needs to be perfect for your app to run as expected, and also needs to match your production environment.

> 在过去，如果你准备开发一个Python应用，你首先要准备安装Python运行环境到服务器。但如此一来，你的开发环境需要完美的匹配应用运行所需要的环境，并且还需要保证生产也是同样的环境。

With Docker, you can just grab a portable Python runtime as an image, no installation necessary. Then, your build can include the base Python image right alongside your app code, ensuring that your app, its dependencies, and the runtime, all travel together.

> 而使用了Docker技术，你可以将一个轻量的Python运行环境放入一个镜像之中，无需多次安装。之后的程序构建以此镜像作为Python基础镜像，应用代码运行在其中，保证了应用与其依赖和运行环境同时被加载运行。

These portable images are defined by something called a Dockerfile.

> 这些轻量便捷的镜像由Dockerfile的文件来定义。

### Define a container with Dockerfile 使用Dockerfile来定义容器

Dockerfile defines what goes on in the environment inside your container. Access to resources like networking interfaces and disk drives is virtualized inside this environment, which is isolated from the rest of your system, so you need to map ports to the outside world, and be specific about what files you want to “copy in” to that environment. However, after doing that, you can expect that the build of your app defined in this Dockerfile behaves exactly the same wherever it runs.

> Dockerfile定义了容器内运行环境中所发生的事件。例如对网络接口和磁盘驱动器等资源的访问，在此环境中进行了虚拟化，并且与系统的其他部分是隔离的。因此，对网络接口来说，你需要将容器内的端口映射到容器外部，对磁盘使用来说，你需要具体指定哪些文件要“复制”到容器环境中。在执行类似操作后，你会发现Dockerfile中所定义的应用行为与在任意位置运行起来的的应用是完全一致的。

#### Dockerfile

Create an empty directory on your local machine. Change directories (cd) into the new directory, create a file called Dockerfile, copy-and-paste the following content into that file, and save it. Take note of the comments that explain each statement in your new Dockerfile.

> 在本地计算机上创建一个空目录。进入新目录，创建名为```Dockerfile```的文件，将以下内容复制并粘贴到该文件中，然后保存。 记住新Dockerfile中每个语句的注释。

```
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

This Dockerfile refers to a couple of files we haven’t created yet, namely app.py and requirements.txt. Let’s create those next.

> 这个Dockerfile引用了我们尚未创建的几个文件，即app.py和requirements.txt。让我们接下来创建它们。

### The app itself 应用自身

Create two more files, requirements.txt and app.py, and put them in the same folder with the Dockerfile. This completes our app, which as you can see is quite simple. When the above Dockerfile is built into an image, app.py and requirements.txt is present because of that Dockerfile’s COPY command, and the output from app.py is accessible over HTTP thanks to the EXPOSE command.

> 再创建两个文件：requirements.txt和app.py，并将它们与Dockerfile放在同一个文件夹中。这就完成了我们的应用程序，可以看到非常简单。当使用上面的Dockerfile打包镜像时，由于Dockerfile的```COPY```命令，app.py和requirements.txt被拷贝到镜像中，并由EXPOSE命令将app.py的输出通过端口暴露给外部通过HTTP访问。

#### requirements.txt

```
Flask
Redis
```

#### app.py

```
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

Now we see that pip install -r requirements.txt installs the Flask and Redis libraries for Python, and the app prints the environment variable NAME, as well as the output of a call to socket.gethostname(). Finally, because Redis isn’t running (as we’ve only installed the Python library, and not Redis itself), we should expect that the attempt to use it here fails and produces the error message.

> 我们看到```pip install -r requirements.txt```为Python安装Flask和Redis库，应用打印出环境变量NAME，以及对```socket.gethostname()```的调用输出。最后，因为Redis没有运行（因为我们只安装了Python库，而没有Redis本身），我们应该能看到它尝试调用失败并产生的错误消息。

Note: Accessing the name of the host when inside a container retrieves the container ID, which is like the process ID for a running executable.

> 在容器内部想要获取容器ID时，请访问主机名称，类似于正在运行的可执行文件的进程ID。

```
root@ebfea45266f3:/# hostname
ebfea45266f3
root@ebfea45266f3:/# env |grep NAME
HOSTNAME=ebfea45266f3
```

That’s it! You don’t need Python or anything in requirements.txt on your system, nor does building or running this image install them on your system. It doesn’t seem like you’ve really set up an environment with Python and Flask, but you have.

> 就是这样，您的系统上不再需要Python运行环境或requirements.txt中的任何安装内容，构建或运行镜像也不需要额外安装。你似乎并没有真正为Python和Flask安装一个运行环境的环境，但事实上你做到了。

### Build the app 构建应用

We are ready to build the app. Make sure you are still at the top level of your new directory. Here’s what ls should show:

> 准备构建应用啦。确认你还在新目录的顶层。下面是执行```ls```后应当显示的内容：

```
$ ls
Dockerfile		app.py			requirements.txt
```

Now run the build command. This creates a Docker image, which we’re going to name using the --tag option. Use -t if you want to use the shorter option.

> 现在执行构建命令。这个命令会创建一个Docker镜像，我们使用```--tag```为其命名（标签）。如果觉得--tag过长，可以使用```-t```这个简短命令替代。

```
docker build --tag=friendlyhello .
docker build -t friendlyhello .
```

Where is your built image? It’s in your machine’s local Docker image registry:

> 去哪里找到新构建的镜像呢？它保存在你服务器的本地Docker镜像仓库中：

```
$ docker image ls

REPOSITORY            TAG                 IMAGE ID
friendlyhello         latest              326387cea398

$ docker images
```

Note how the tag defaulted to latest. The full syntax for the tag option would be something like --tag=friendlyhello:v0.0.1.

> 记住tag标签默认会使用latest。完整的标签像是这样：```tag=friendlyhello:v0.0.1```，没有```v0.0.1```的话就成了默认的```tag=friendlyhello:latest```


**Troubleshooting for Linux users**  
***Proxy server settings***

Proxy servers can block connections to your web app once it’s up and running. If you are behind a proxy server, add the following lines to your Dockerfile, using the ENV command to specify the host and port for your proxy servers:

> 代理服务器可以在Web应用程序启动并运行后阻止其连接。如果你服务器位于代理后面，请将以下内容添加到Dockerfile，使用ENV命令指定代理服务器的主机和端口：

```
# Set proxy server, replace host:port with values for your servers
ENV http_proxy host:port
ENV https_proxy host:port
```

***DNS settings***

DNS misconfigurations can generate problems with pip. You need to set your own DNS server address to make pip work properly. You might want to change the DNS settings of the Docker daemon. You can edit (or create) the configuration file at /etc/docker/daemon.json with the dns key, as following:

> DNS的配置错误可能会导致pip安装出现问题。你需要设置自己的DNS服务器地址才能使pip正常工作。你可以直接更改Docker守护程序的DNS设置，编辑（或创建）配置文件```/etc/docker/daemon.json```，更新dns内容如下所示：

```
{
  "dns": ["your_dns_address", "8.8.8.8"]
}
```

In the example above, the first element of the list is the address of your DNS server. The second item is Google’s DNS which can be used when the first one is not available.

> 在上面的例子中，列表里的第一个元素是你的DNS服务器地址，第二个元素是Google公司的DNS地址，当第一个自定义的DNS不可用或解析失败时会使用第二个Google的DNS服务。

Before proceeding, save daemon.json and restart the docker service.

> 继续运行之前，保存```daemon.json```并重启docker服务

```
sudo service docker restart
```

Once fixed, retry to run the build command.

> 当配置固定下来之后，重新执行打包构建命令

### Run the app 运行应用

Run the app, mapping your machine’s port 4000 to the container’s published port 80 using -p:

> 运行应用，将服务器的4000端口映射至容器对外暴露的80端口，使用-p命令：

```
docker run -p 4000:80 friendlyhello
```

You should see a message that Python is serving your app at http://0.0.0.0:80. But that message is coming from inside the container, which doesn’t know you mapped port 80 of that container to 4000, making the correct URL http://localhost:4000.

> 你应当看到一条Python启动服务http://0.0.0.0:80的日志信息。但是信息来自容器内部，其并不知道你已经把容器的80端口映射到了服务器的4000端口，所以应当使用正确的地址访问服务：

```
http://localhost:4000
```

Go to that URL in a web browser to see the display content served up on a web page.

> 在浏览器中输入上面的URL可以看到网页上显示的内容

Note: If you are using Docker Toolbox on Windows 7, use the Docker Machine IP instead of localhost. For example, ```http://192.168.99.100:4000/```. To find the IP address, use the command docker-machine ip.

> 如果您在Windows 7上使用Docker Toolbox，请使用Docker Machine IP替换localhost。例如，```http://192.168.99.100:4000/```。要查找IP地址，请使用命令docker-machine ip。

You can also use the curl command in a shell to view the same content.

> 你也可以在shell中使用```curl```命令查看到相同的内容

```
$ curl http://localhost:4000

<h3>Hello World!</h3><b>Hostname:</b> 8fc990912a14<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
```

This port remapping of 4000:80 demonstrates the difference between EXPOSE within the Dockerfile and what the publish value is set to when running docker run -p. In later steps, map port 4000 on the host to port 80 in the container and use http://localhost.

> 此次的端口映射(4000:80)示范了Dockerfile中的EXPOSE暴露端口和运行```docker run -p```使用的公布端口之间的区别（EXPOSE主要是提示作用，表明容器向外暴露的端口是80，docker run -p是实际的映射作用，把打到服务器的4000端口的请求转发到了容器的80端口。即使没有EXPOSE的提示，上面的命令执行后依然得到相同的显示结果）。在后面的步骤中，映射服务器端口4000到容器的80端口，并使用```http://localhost```

Hit CTRL+C in your terminal to quit.

> 使用CTRL+C命令推出当前终端

On Windows, explicitly stop the container  
On Windows systems, CTRL+C does not stop the container. So, first type CTRL+C to get the prompt back (or open another shell), then type docker container ls to list the running containers, followed by docker container stop <Container NAME or ID> to stop the container. Otherwise, you get an error response from the daemon when you try to re-run the container in the next step.

> Windows系统中明确地结束容器进程  
> 对于Windows系统，```CTRL+C```并不是结束容器进程。首先，运行```CTRL+C```快速返回（或打开另一个终端），接着运行```docker container ls```列出正在运行的容器，再继续运行```docker container stop <Container NAME or ID>```结束容器。否则，当你在后续步骤中试图重新运行一个容器时，会得到后台进程返回的错误响应。

Now let’s run the app in the background, in detached mode:

> 现在让我们在后台运行应用，使用离线模式：

```
docker run -d -p 4000:80 friendlyhello
```

You get the long container ID for your app and then are kicked back to your terminal. Your container is running in the background. You can also see the abbreviated container ID with docker container ls (and both work interchangeably when running commands):

> 你可以看到执行命令后返回的应用长ID，并且当前被踢出了容器的终端。容器现在是运行在后台。你也可以使用```docker container ls```命令查看容器的简短ID（都是有效的容器ID）

```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED
1fa4ab2cf395        friendlyhello       "python app.py"     28 seconds ago
$ docker ps 
$ docker ps -a
```

Notice that CONTAINER ID matches what’s on ```http://localhost:4000```.

> 记住容器ID对应的就是```http://localhost:4000```后面所运行的容器

Now use docker container stop to end the process, using the CONTAINER ID, like so:

> 现在使用```docker container sto```结束容器进程，需要容器ID，如：

```
docker container stop 1fa4ab2cf395
```

### Share your image 共享镜像

To demonstrate the portability of what we just created, let’s upload our built image and run it somewhere else. After all, you need to know how to push to registries when you want to deploy containers to production.

> 为了演示上面工作内容的可移植性，上传我们构建的镜像并在其他任意地方运行。毕竟，当你想要将容器部署到生产环境时，需要学习如何推送镜像到镜像仓库。

A registry is a collection of repositories, and a repository is a collection of images—sort of like a GitHub repository, except the code is already built. An account on a registry can create many repositories. The docker CLI uses Docker’s public registry by default.

> 仓库是镜像库的集合，镜像库是类似GitHub仓库的的一类镜像的集合，不同的是代码已经构建完成。仓库上的帐户可以创建许多镜像库。 docker CLI默认使用Docker的公共仓库（hub.docker.com）。

Note: We use Docker’s public registry here just because it’s free and pre-configured, but there are many public ones to choose from, and you can even set up your own private registry using Docker Trusted Registry.

> 注意：我们在这里使用Docker公用仓库是因为其免费并且已做了预配置，实际上有很多公用仓库都可以选择，甚至你可以安装自己私有的仓库，详见[Docker Trusted Registry](https://docs.docker.com/datacenter/dtr/2.2/guides/)

#### Log in with your Docker ID 使用DockerID登陆容器仓库

If you don’t have a Docker account, sign up for one at hub.docker.com. Make note of your username.

Log in to the Docker public registry on your local machine.

> 如果你没有Docker账号，请在hub.docker.com上注册。记住你的用户名。  
> 在本地服务器上登陆Docker公共仓库：

```
$ docker login
$ docker login -u xx -p xx registry.xx.com（私有仓库）
```

#### Tag the image 镜像标签

The notation for associating a local image with a repository on a registry is username/repository:tag. The tag is optional, but recommended, since it is the mechanism that registries use to give Docker images a version. Give the repository and tag meaningful names for the context, such as get-started:part2. This puts the image in the get-started repository and tag it as part2.

> 本地镜像与仓库上镜像相关联的标记方法是```username/repository:tag```。 虽然标签是可选的，但强烈建议使用标签，因为它是仓库用来为Docker镜像提供版本控制的机制。为镜像设置有意义的名称和标签，例如```get-started:part2```，这会将镜像放入```get-started```镜像库并将其标记为```part2```。

Now, put it all together to tag the image. Run docker tag image with your username, repository, and tag names so that the image uploads to your desired destination. The syntax of the command is:

> 现在，把上面所有内容放在一起打包镜像。执行```docker tag image```带上用户名，镜像名和标签，这样上传后就能放到正确的位置。命令格式如下：

```
docker tag image username/repository:tag
```

例如：

```
docker tag friendlyhello gordon/get-started:part2
```

Run docker image ls to see your newly tagged image.

> 运行```docker image ls```查看新打包的镜像

```
$ docker image ls

REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
friendlyhello            latest              d9e555c53008        3 minutes ago       195MB
gordon/get-started         part2               d9e555c53008        3 minutes ago       195MB
python                   2.7-slim            1c7128a655f6        5 days ago          183MB
...
```

#### Publish the image 上传公开镜像

Upload your tagged image to the repository:

> 上传已标记的镜像至仓库：

```
docker push username/repository:tag
```

Once complete, the results of this upload are publicly available. If you log in to Docker Hub, you see the new image there, with its pull command.

> 完成之后，上传的结果是镜像变成公开可用的。如果登陆了Docker Hub仓库，你会在那看到新的镜像，并且有pull下拉镜像的提示命令。

#### Pull and run the image from the remote repository 从远程仓库下载并运行镜像

From now on, you can use docker run and run your app on any machine with this command:

> 从现在开始，你可以使用```docker run```命令在任意服务器运行你的应用：

```
docker run -p 4000:80 username/repository:tag
```

If the image isn’t available locally on the machine, Docker pulls it from the repository.

> 如果在本地服务器上镜像不可用，Docker会从远程仓库拉取。

```
$ docker run -p 4000:80 gordon/get-started:part2
Unable to find image 'gordon/get-started:part2' locally
part2: Pulling from gordon/get-started
10a267c67f42: Already exists
f68a39a6a5e4: Already exists
9beaffc0cf19: Already exists
3c1fe835fb6b: Already exists
4c9f1fa8fcb8: Already exists
ee7d8f576a14: Already exists
fbccdcced46e: Already exists
Digest: sha256:0601c866aab2adcc6498200efd0f754037e909e5fd42069adeff72d1e2439068
Status: Downloaded newer image for gordon/get-started:part2
 * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
```

No matter where docker run executes, it pulls your image, along with Python and all the dependencies from requirements.txt, and runs your code. It all travels together in a neat little package, and you don’t need to install anything on the host machine for Docker to run it.

> 不论```docker run```在哪里执行，它都会拉取你的镜像，连带着Python环境和所有requirements.txt中的依赖，并运行你的代码。所有的内容都会在一个整洁小巧的包文件里，你不必为Docker而在服务器上安装任何内容。

### Conclusion of part two 第二部分总结

That’s all for this page. In the next section, we learn how to scale our application by running this container in a service.

> 以上是本部分的所有内容。在下一个章节，我们会学习如何通过service服务中的容器来扩展我们的应用。

### Recap and cheat sheet (optional) 回顾和总结

Here is a list of the basic Docker commands from this page, and some related ones if you’d like to explore a bit before moving on.

> 以下是关于Docker基本命令的列表，还包括一些相关进阶命令，如果你有兴趣在后续章节进行之前去探索的话。

```
docker build -t friendlyhello .  # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyhello  # Run "friendlyname" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyhello         # Same thing, but in detached mode
docker container ls                                # List all running containers
docker container ls -a             # List all containers, even those not running
docker container stop <hash>           # Gracefully stop the specified container
docker container kill <hash>         # Force shutdown of the specified container
docker container rm <hash>        # Remove specified container from this machine
docker container rm $(docker container ls -a -q)         # Remove all containers
docker image ls -a                             # List all images on this machine
docker image rm <image id>            # Remove specified image from this machine
docker image rm $(docker image ls -a -q)   # Remove all images from this machine
docker login             # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag  # Tag <image> for upload to registry
docker push username/repository:tag            # Upload tagged image to registry
docker run username/repository:tag                   # Run image from a registry
```



