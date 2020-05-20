# docker学习  
当安装docker的时候会涉及到两个主要的组件：docker客户端和docker daemon(也称为服务端或者docker引擎)，其中docker daemon实现了docker引擎的API。在linux环境下安装的时候，客户端和daemon的通信时通过IPC/Unix socket实现的（/var/run/docker.sock）  
## docker引擎  
docker引擎主要由以下组件组成：docker客户端（docker client），docker守护进程（docker daemon），containerd和runc。它们共同服务docker的创建和运行  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/05/18/1589770980912-1589770980916.png)  
目前docker引擎的结构如下：  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/05/18/1589771328497-1589771328505.png)  
启动容器的流程：  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/05/18/1589771376114-1589771376115.png)  
## docker镜像  
docker镜像就像停止运行的容器  
使用docker镜像首先可以从docker仓库中拉取镜像，常用的镜像仓库时docker hub，也可以是其他的  
镜像由多层组成，每层叠加之后从外部看起来就像一个整体。镜像的内部是一个精简的os，只包含应用运行时所必须的文件和依赖包  
镜像可以理解为是一种构建时结构，容器可以理解为是一种运行时结构
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/05/18/1589771904783-1589771904788.png)  
一旦容器从镜像启动以后，他们二者之间的关系就变成了相互依赖的关系，并且在容器停止之前不能删除镜像。如果强制删除镜像，会导致容器的异常  
docker客户端的镜像仓库服务是可以配置的，默认使用docker hub。镜像仓库服务包含多个镜像仓库，每个仓库又包含多个镜像。
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/05/18/1589772225340-1589772225342.png)  
只要给出镜像名字和标签，就可以在镜像仓库中定位一个镜像。镜像名称和标签之间使用":"分割  
```shell
docker imgae pull <repository>:<tag>
```
没有指定镜像标签默认会使用latest标签，但是latest标签并不意味着镜像是最新的  
可以使用如下命令是过滤返回的镜像的内容
```shell
docker images filter(docker images -f )
```

filter 支持以下过滤器  
\-\-\-dangling，返回虚悬镜像。true表示仅返回虚悬镜像，false表示不返回虚悬镜像  
\-\-\-before，需要镜像名称或者ID，返回在此之前创建的所有镜像  
\-\-\-after，需要镜像名称或者ID，返回在此之后创建的所有镜像  
\-\-\-label，根据镜像的标注或者值对镜像进行过滤，输出的内容不包含所标注的值  
`docker search`允许使用命令行的方式搜索镜像，默认情况下只返回25行内容，可以使用\-limit参数返回更多的行，最多可以返回100行  
docker镜像由一些只读层的镜像组成，docker负责将这些曾堆叠起来，对外表示为单个统一的整体  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/05/18/1589774182697-1589774182700.png)  
可以通过`docker image inspect`命令查看docker镜像的所有分层  
当在不需要docker镜像时，可以通过`docker image rm`命令删除docker镜像。如果某个镜像层被多个镜像所共享，那么需要在所有依赖该镜像层的镜像被删除之后才能删除该镜像层。并且在该镜像层上又正在运行的容器的时候，删除该镜像层是不被允许的。如果强行删除会引发问题。或者删除镜像需要运行在该镜像层上的所有容器都停止运行以后才可以  
## docker容器（container）  
docker和虚拟机最大的区别就是容器更快，并且更轻量级，虚拟机运行在完整的操作系统只上，而容器则是共享其主机的操作系统或内核  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/05/18/1589789149067-1589789149093.png)  
docker镜像可以运行多个容器  
启动容器的命令是`docker container run <image> <app>` 指定容器启动时所使用的镜像和在容器中运行的应用，启动容器还有其他的参数可以使用  
\-it 可以连接当前终端到容器的shell终端  
\-d 以后台形式运行一个容器
\-p 端口映射，主机端口:容器端口
\-v 目录映射，主机目录:容器目录
`docker container ls`可以列出所有正在运行的容器  
`docker exec -it container-id /bin/bash`可以用于重新连接正在运行的容器  
建议在容器运行时配置好重启策略，可以在指定事件发生或者错误发生后重启容器  
重启策略应用于每个容器，可以作为参数传入`docker container run`命令或者compose文件中，docker重启支持以下三种策略：  
always策略，是一种简单的策略，除非明确使用`docker container stop`命令停止的容器，否则会不停的尝试重启处于停止状态的容器    
unless-stopped策略与always策略很类似，区别在于always会在重启docker daemon时也会重启docker，而unless-stopped则不会  
on-failed会重启退出容器时返回状态不为0的容器  
## docker容器常用命令  
`docker container run `启动新容器的命令，该命令最简单的形式就是接收一个镜像和命令作为参数。镜像用于创建容器，命令则是希望运行在容器中的应用  
`docker container ls`列出所有正在运行的容器  
`docker container exec`用于正在运行的容器，启动一个新进程。该命令最常用的形式是`docker exec -it <container-id orcontainer name> /Bash`，使用该命令会在容器内部启动一个Bash shell 进程用于连接。但是使用该命令有一个前提就是容器内部必须包含Bash shell  
`docker container stop`用于停止一个正在运行的容器，并将状态置为Exited(0),该命令是通过发送SIGTERN信号给容器内的PID为1的进程达到目的。如果进程没有在10秒钟内清理并停止运行，那么会接着发送SIGKILL信号来强制停止该容器  
`docker container start`用于启动处于停止状态（Exited）的容器  
`docker container rm`删除停止运行的容器  
`docker container inspect`显示容器的详细运行时细节和配置信息  
## docker应用容器化  
docker能够简化应用的构建，部署和运行过程。完整的应用容器化主要包含以下几个步骤：  
1.编写应用代码  
2.创建一个Dockerfile，其中包含当前应用的描述，依赖以及如何运行这个应用  
3.对该Dockerfile执行`docker image build .`命令  
4.等待docker将应用构建到其中  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/05/19/1589870434743-1589870434790.png)  
在Docker中，将包含应用文件的目录通常称为构建上下文，，通常将Dockerfile放到构建目录中。另外一点很重要就是文件名称一定是Dockerfile，不能是dockerfile或者Docker file  
Dockerfile主要有两个功能  
对当前应用的描述  
指导docker完成应用的容器化  
Dockerfile由一行行命令组成，并且支持以#开头的注释  
一般的Dockerfile分为四部分，基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令
例如：  
```shell
# This dockerfile uses the ubuntu image
# VERSION 2 - EDITION 1
# Author: docker_user
# Command format: Instruction [arguments / command] ..

# Base image to use, this must be set as the first line
FROM ubuntu

# Maintainer: docker_user <docker_user at email.com> (@docker_user)
MAINTAINER docker_user docker_user@email.com

# Commands to update the image
RUN echo "deb http://archive.ubuntu.com/ubuntu/ raring main universe" >> /etc/apt/sources.list
RUN apt-get update && apt-get install -y nginx
RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf

# Commands when creating a new container
CMD /usr/sbin/nginx

```  
一开始必须指明所基于的镜像名称，接下来推荐说明维护者信息后面则是镜像的操作命令，最后是CMD指令，来指定容器运行时的操作命令  
### 指令  
指令的格式一般为`instruction arguments`，指令包括FROM、RUN、CMD等
#### FROM  
格式为`FROM <image>` 或者`FROM <imgea>:<tag>`  
第一条命令必须是FROM指令，并且如果在同一个Dockerfile中创建多个镜像时，可以使用多个FROM指令（每个镜像一次）  
#### MAINTAINER  
格式为`MAINTAINER <name>`,指定维护者信息  
#### RUN  
格式为`RUN <command>`或者`RUN ["executable"，"parm1","parm2"]`  
前者使用shell终端运行命令，即`/bin/bash -c`；后者则使用`exec`来执行。指定其他终端可以使用第二种方式实现，例如`RUN ["/bin/bash","-c","echo hello"]`  
每条RUN指令都会在当前镜像的基础上执行制定命令，并且提交为新的镜像。当命令较长时可以使用`\`来换行  
在撰写Dockerfile的时候也时刻提醒自己不是在写shell，而是在定义每一层该如何构建，在每一层构建完成之后一定要记得清理掉无关文件。因此在构建镜像时一定要确保每一层只添加了真正需要的东西，无关紧要的东西都应该被清理掉。  
#### CMD  
支持三种格式  
`CMD ["executable","parm1","parm2"]`使用`exec`执行，推荐这种方式。这类格式在解析时会解析成JSON数组，因此一定要使用双引号`"`,而不要使用单引号`'`
`CMD command parm1 parm2`在`/bin/sh`中执行，提供给需要交互的应用  
`CMD ["parm1","parm2"]`提供给`ENTRYPOINT`的默认参数  
指定启动容器时执行的命令，每个Dockerfile只能有一个CMD命令，如果由多个只有最后一个会执行。如果用户启动容器时制定了运行时的指令，则会覆盖掉CMD指定的命令  
应该要注意docker和虚拟机之间的区别，docker中没有后台的概念，所以在docker中的所有应用都应该使用前台的方式来执行。对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，一旦主进程退出，容器就是去了存在的意义，从而退出。例如在docker中启动nginx应该使用如下命令：  
`CMD ["nginx","-g","daemon off"]`  
#### EXPOSE  
格式为`EXPOSE <port> [<port>...]`  
告诉服务端容器docker暴露的端口号，共互联使用。在启动容器是需要使用-P参数，docke主机会自动分配一个端口转发到指定的端口  
#### ENV  
格式为`ENV <key> <value>` 指定一个环境变量，会被后续`RUN`使用，并在容器运行时保持  
#### ADD  
格式为`ADD <src> <dest>` 该命令将复制指定的`src`到容器`dest`。其中`src`可以是Dockerfile所在目录的一个相对路径，也可以URL，还可以是一个tar文件（自动解压为目录）  
#### COPY  
格式为`COPY [--chown=<user>:<group>] <src> <dest>`或者`COPY [--chown=<user>:<group>] <src1>... <dest>`复制本地主机的`src`（Dockerfile目录的相对路径)容器的`dest`,当使用本地目录为源目录时，推荐使用`COPY`  
#### ENTRYPOINT  
两种格式  
`ENTRYPOINT ["executalbe","parm1","parm2"]`  
`ENTRYPOINT command parm1 parm2` (shell)中执行  
配置容器启动后的命令并且不可以被`docker run`提供的参数覆盖，每个Dockerfile中只能有一个ENTRYPOINT,当制定多个时，只有最后一个会生效  
`ENTRYPOINT`可以在运行时被替代，使用`docker run --extrypoint`参数来指定。如果Dockerfile指定了`ENTRYPOINT`,那么`CMD`的含义就发生了变化，不再是直接的运行其命令，而是将`CMD`内容作为参数传递给`ENTRYPOINT`，即实际的执行会变为：  
`<ENTRYPOINT> <CMD>`  
#### VOLUME  
格式为`VOLUME ["/data"]`  
创建一个可从本地主机或者其他容挂载的挂载点，一般用来存放数据库和需要保持的数据等  
这里/data目录会在运行时自动挂载为匿名卷，任何向/data写入的信息都不会记录进容器的存储层。当然运行时也可以覆盖这个挂载设置。
`docker run -d -v mydata:/data xxx`这行命令就会使用mydata这个命名卷挂载到/data这个位置，替代了Dockerfile中的匿名挂在卷  
#### USER  
格式为`USER daemon`  
指定运行容器的用户名或者UID，后续的RUN命令也会使用这个用户。当然这个命令只是帮你切换到用户名而已，因此这个用户必须是事先创建好的，否则无法切换  
如果要临时获取管理员权限可以使用`gosu`，而不推荐使用`sudo`  
####WORKDIR  
格式为`WORKDIR /path/to/workdir`  
以后各层的当前目录就会被该为指定的目录。例如：  
```shell  
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```  
最终的路径为/a/b/c  
#### HEALTHCHECK  
`HEALTHCHECK [选项] CMD <命令>`设置检查容器健康状态的命令  
`HEALTHCHECK NONE`如果基础镜像有健康状态检测，这条命令会屏蔽掉基础镜像的健康状态检查命令  
`HEALTHCHECK`是告诉docker如何判断该容器的状态是否正常  
当一个镜像指定了`HEALTHCHECK`指令后，用其启动容器，初始状态会变为`starting`,在`HEALTHCHECK`指令执行成功后变为`healthy`状态，如果连续一定次数失败，则会变为`unhealthy`状态  
可以包含以下参数：  
`--interval=<间隔>`:两次健康检查的时间间隔，默认是30秒  
`timeout=<时长>`：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就视为失败，默认30秒  
`retries=<>`






