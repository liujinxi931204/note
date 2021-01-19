# docker学习  
当安装docker的时候会涉及到两个主要的组件：docker客户端和docker daemon(也称为服务端或者docker引擎)，其中docker daemon实现了docker引擎的API。在linux环境下安装的时候，客户端和daemon的通信时通过IPC/Unix socket实现的（/var/run/docker.sock）  
## docker引擎  
docker引擎主要由以下组件组成：docker客户端（docker client），docker守护进程（docker daemon），containerd和runc。它们共同服务docker的创建和运行  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/18/1589770980912-1589770980916.png)  
目前docker引擎的结构如下：  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/18/1589771328497-1589771328505.png)  
启动容器的流程：  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/18/1589771376114-1589771376115.png)  

## docker镜像  
docker镜像就像停止运行的容器  
使用docker镜像首先可以从docker仓库中拉取镜像，常用的镜像仓库时docker hub，也可以是其他的  
镜像由多层组成，每层叠加之后从外部看起来就像一个整体。镜像的内部是一个精简的os，只包含应用运行时所必须的文件和依赖包  
镜像可以理解为是一种构建时结构，容器可以理解为是一种运行时结构
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/18/1589771904783-1589771904788.png)  
一旦容器从镜像启动以后，他们二者之间的关系就变成了相互依赖的关系，并且在容器停止之前不能删除镜像。如果强制删除镜像，会导致容器的异常  
docker客户端的镜像仓库服务是可以配置的，默认使用docker hub。镜像仓库服务包含多个镜像仓库，每个仓库又包含多个镜像。
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/18/1589772225340-1589772225342.png)  
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
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/18/1589774182697-1589774182700.png)  
可以通过`docker image inspect`命令查看docker镜像的所有分层  
当在不需要docker镜像时，可以通过`docker image rm`命令删除docker镜像。如果某个镜像层被多个镜像所共享，那么需要在所有依赖该镜像层的镜像被删除之后才能删除该镜像层。并且在该镜像层上又正在运行的容器的时候，删除该镜像层是不被允许的。如果强行删除会引发问题。或者删除镜像需要运行在该镜像层上的所有容器都停止运行以后才可以  
## docker容器（container）  
docker和虚拟机最大的区别就是容器更快，并且更轻量级，虚拟机运行在完整的操作系统只上，而容器则是共享其主机的操作系统或内核  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/18/1589789149067-1589789149093.png)  
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
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/19/1589870434743-1589870434790.png)  
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
#### WORKDIR  
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
`--timeout=<时长>`：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就视为失败，默认30秒  
`--retries=<次数>`:当连续失败指定次数之后，则将容器的状态置为`unhealthy`，默认3次  
`HEALTHYCHECK`和`CMD`还有`ENTRYPOINT`一样只可以出现一次，如果出现多次，则最后一次有效  
假设有个最简单的WEB服务，可以通过使用curl来帮助判断服务的健康性，其Dockerfile的HEALTHYCHECK可以这样写  
```shell
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=5s --timeout=3s \
  CMD curl -fs http://localhost/ || exit 1
```
## docker compose
Docker compose与docker stack很类似，能够在docker节点上，以单引擎(single-engine mode )的方式进行多容器的部署，docker compose并不是用过脚本或者冗长docker命令将应用组织起来，而是通过一个声名式的配置文件描述整个应用，从而使用一条命令完成部署  
他允许用户通过一个单独的`docker-compose.yml`模板文件(YAML)来定义一组相关联的应用容器为一个项目(project)  
`compose`中有两个重要的概念：  
服务(service)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例  
项目(project)：有一组关联的应用容器组成的一个完整的业务单元，在`docker-compose.yml`文件中定义  
一个项目可以由多个服务(容器)关联而成，compose面向项目进行管理  
官方提供了一个docker-compose配置文件的标准例子  
```shell
version: "3"
services:
 
  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
 
  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]
 
  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - 5000:80
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure
 
  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - 5001:80
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
 
  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]
 
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
 
networks:
  frontend:
  backend:
 
volumes:
  db-data:
```
一份标准的配置文件应该包含version、service、networks三大部分，其中最关键的就是service和netwos  
## 配置选项  
#### build  
服务除了可以基于指定的镜像，还可以基于一份Dockerfile，在使用up启动之时执行构建任务，这个构建标签就是build，它可以指定Dockerfil所在文件夹路径。Compose将会利用它自动构建这个镜像，然后使用这个镜像启动服务容器  
```shell
build:/path/to/build/dir
```
也可以是相对路径  
```shell
build:./dir
```
设定上下文根目录，然后以该目录为准指定Dockerfile  
```shell
build:
    context: ../
    dockerfile: path/of/Dockerfile
```
#### context  
conetxt选项可以是Dockerfile的文件路径，也可以是连接到git仓库的url，当提供的值是相对路径时，它被解析为相对于撰写文件的路径，此目录也是发送到Docker守护进程的context  
```shell
build:
    conetxt: ./dir
```
#### dockerfile  
使用词Dockerfile文件来构建，必须指定构建路径  
```shell
build:
    context: .
    dockerfile: Dockerfile-alternate
```
#### image  
```shell 
services:
    web:
        image:nginx
```
在services标签下的第二级标签是web，这个名字是用户自己定义的，用来指定服务的名称  
image则是指定服务的镜像名称或者镜像ID。如果镜像在本地不存在，compose则会尝试拉取这个镜像  
例如：  
```shell
image: redis
image: ubuntu:14.04
image: tutum/influxb
image: a4bc65fd
```
#### args  
添加构建参数，这些参数是仅在构建过程中可以访问的环境变量  
首先在Dockerfile中指定参数  
```shell
ARG fendo
ARG password

RUN echo "build number: $fendo"
RUN script-requiring-password.sh "$password"
```
然后指定build下的参数，可以传递映射列表  
```shell
build:
    context:
    args:
        fendo:1
        password:fendo
```
#### command  
使用command可以覆盖容器启动后默认的执行命令  
```shell
command:build exec thin -p 3000
```
该命令也可以是一个列表，方法类似于Dockerfile  
```shell

command:["build","exec","thin","-p","3000"]
```
#### container_name  
compose的容器命名格式是:<项目名称> <服务名称><序号>  
虽然可以自定义项目名称、服务名称，但是如果想完全控制容器的名称，可以使用这个标签  
```shell
container_name:app
```
这样容器的名字就指定为app了  
#### depends_on  
一般项目中启动容器的顺序是有要求的，deponds_on就是为了解决容器之间依赖关系，启动先后顺序的问题的  
```shell
version: '3'
services:
    web:
        build: .
        deponds_on:
            - db
            - redis
    redis:
        image: redis
    db:
        image: postgres
```
需要注意的是，默认情况下使用docker-compose up web 这样的方式情动web服务时，也会启动redis和db这两个服务，因为在配置文件中定义了依赖关系  
#### pid  
```shell
pid: "host"
```
将pid模式设置为主机pid模式，跟主机系统共享进程命名空间。容器使用这个标签能够访问和操纵其他容器和宿主机的命名空间  
#### port  
端口映射的标签  
使用HOST:CONTAINER格式或者只是指定容器的端口，宿主机会随机映射端口  
```shell
ports:
    - "3000"
    - "8000:8000"
    - "49100:22"
    - "127.0.0.1:8001:8001"
```
#### extra_hosts  
添加主机名的标签，就是往/etc/hosts文件中添加一些记录，与Docker client的--add-host类似  
```shell
extra_hosts:
    - "somehost:162.242.195.82"
    - "otherhost:50.31.209.229"
```
启动之后查看容器内部的hosts:
```shell
162.242.195.82  somehost
50.31.209.229  otherhost
```
#### volumes  
数据卷的格式可以是以下多种形式：
```shell
voulmes:
    //只是指定一个路径，docker会自动创建一个数据卷（这个路径是容器内部的）
    - /var/lib/mysql
    //使用绝对路径挂载数据卷
    - /opt/data:/var/lib/mysql
    //以compose配置文件为中心的相对路径作为数据卷挂载到容器
    - ./cache:/tmp/cache
    //使用用户的相对路径（~/表示的目录是 /home/<用户目录> 或者 /root/）
    - ~/configs:/etc/configs/:ro
    //已经存在的命名数据卷
    - dataVolume:/var/lib/mysql
```
如果你不使用宿主机路径，可以指定一个volume_driver  
```shell
volume_driver: mydirver
```
#### links  
链接到另一个服务中的容器。请指定服务名称和链接别名（services：alias），或者仅指定服务名称
```shell
web:
    links:
        - db
        - db:databases
        - redis
```
在当前的web服务的容器中可以通过链接的db服务的别名database访问db容器中的数据应用，如果没有指定别名，则可以直接使用服务名访问  
链接不需要启用服务进行通信，默认情况下，任何服务都可以以该服务的名称达到任何其他服务。links也可以起到和deponds_on相似的功能，即定义服务之间的依赖关系，从而确定服务的启动顺序  
#### external_links  
链接到docker-compose.yml之外的容器，甚至并非compose管理的容器，参数格式类似于links
```shell
external_links:
    - redis
    - project_db_1:mysql
    - project_db_1:postgresql
```

#### dns  
配置dns服务，可以是一个值也可以是一个列表  
```shell
dns:8.8.8.8
dns:
    - 8.8.8.8
    - 9.9.9.9
```
附docker-compose.ym文件详解  
```shell
Compose和Docker兼容性：
    Compose 文件格式有3个版本,分别为1, 2.x 和 3.x
    目前主流的为 3.x 其支持 docker 1.13.0 及其以上的版本

常用参数：
    version           # 指定 compose 文件的版本
    services          # 定义所有的 service 信息, services 下面的第一级别的 key 既是一个 service 的名称

        build                 # 指定包含构建上下文的路径, 或作为一个对象，该对象具有 context 和指定的 dockerfile 文件以及 args 参数值
            context               # context: 指定 Dockerfile 文件所在的路径
            dockerfile            # dockerfile: 指定 context 指定的目录下面的 Dockerfile 的名称(默认为 Dockerfile)
            args                  # args: Dockerfile 在 build 过程中需要的参数 (等同于 docker container build --build-arg 的作用)
            cache_from            # v3.2中新增的参数, 指定缓存的镜像列表 (等同于 docker container build --cache_from 的作用)
            labels                # v3.3中新增的参数, 设置镜像的元数据 (等同于 docker container build --labels 的作用)
            shm_size              # v3.5中新增的参数, 设置容器 /dev/shm 分区的大小 (等同于 docker container build --shm-size 的作用)

        command               # 覆盖容器启动后默认执行的命令, 支持 shell 格式和 [] 格式

        configs               # 不知道怎么用

        cgroup_parent         # 不知道怎么用

        container_name        # 指定容器的名称 (等同于 docker run --name 的作用)

        credential_spec       # 不知道怎么用

        deploy                # v3 版本以上, 指定与部署和运行服务相关的配置, deploy 部分是 docker stack 使用的, docker stack 依赖 docker swarm
            endpoint_mode         # v3.3 版本中新增的功能, 指定服务暴露的方式
                vip                   # Docker 为该服务分配了一个虚拟 IP(VIP), 作为客户端的访问服务的地址
                dnsrr                 # DNS轮询, Docker 为该服务设置 DNS 条目, 使得服务名称的 DNS 查询返回一个 IP 地址列表, 客户端直接访问其中的一个地址
            labels                # 指定服务的标签，这些标签仅在服务上设置
            mode                  # 指定 deploy 的模式
                global                # 每个集群节点都只有一个容器
                replicated            # 用户可以指定集群中容器的数量(默认)
            placement             # 不知道怎么用
            replicas              # deploy 的 mode 为 replicated 时, 指定容器副本的数量
            resources             # 资源限制
                limits                # 设置容器的资源限制
                    cpus: "0.5"           # 设置该容器最多只能使用 50% 的 CPU 
                    memory: 50M           # 设置该容器最多只能使用 50M 的内存空间 
                reservations          # 设置为容器预留的系统资源(随时可用)
                    cpus: "0.2"           # 为该容器保留 20% 的 CPU
                    memory: 20M           # 为该容器保留 20M 的内存空间
            restart_policy        # 定义容器重启策略, 用于代替 restart 参数
                condition             # 定义容器重启策略(接受三个参数)
                    none                  # 不尝试重启
                    on-failure            # 只有当容器内部应用程序出现问题才会重启
                    any                   # 无论如何都会尝试重启(默认)
                delay                 # 尝试重启的间隔时间(默认为 0s)
                max_attempts          # 尝试重启次数(默认一直尝试重启)
                window                # 检查重启是否成功之前的等待时间(即如果容器启动了, 隔多少秒之后去检测容器是否正常, 默认 0s)
            update_config         # 用于配置滚动更新配置
                parallelism           # 一次性更新的容器数量
                delay                 # 更新一组容器之间的间隔时间
                failure_action        # 定义更新失败的策略
                    continue              # 继续更新
                    rollback              # 回滚更新
                    pause                 # 暂停更新(默认)
                monitor               # 每次更新后的持续时间以监视更新是否失败(单位: ns|us|ms|s|m|h) (默认为0)
                max_failure_ratio     # 回滚期间容忍的失败率(默认值为0)
                order                 # v3.4 版本中新增的参数, 回滚期间的操作顺序
                    stop-first            #旧任务在启动新任务之前停止(默认)
                    start-first           #首先启动新任务, 并且正在运行的任务暂时重叠
            rollback_config       # v3.7 版本中新增的参数, 用于定义在 update_config 更新失败的回滚策略
                parallelism           # 一次回滚的容器数, 如果设置为0, 则所有容器同时回滚
                delay                 # 每个组回滚之间的时间间隔(默认为0)
                failure_action        # 定义回滚失败的策略
                    continue              # 继续回滚
                    pause                 # 暂停回滚
                monitor               # 每次回滚任务后的持续时间以监视失败(单位: ns|us|ms|s|m|h) (默认为0)
                max_failure_ratio     # 回滚期间容忍的失败率(默认值0)
                order                 # 回滚期间的操作顺序
                    stop-first            # 旧任务在启动新任务之前停止(默认)
                    start-first           # 首先启动新任务, 并且正在运行的任务暂时重叠

            注意：
                支持 docker-compose up 和 docker-compose run 但不支持 docker stack deploy 的子选项
                security_opt  container_name  devices  tmpfs  stop_signal  links    cgroup_parent
                network_mode  external_links  restart  build  userns_mode  sysctls

        devices               # 指定设备映射列表 (等同于 docker run --device 的作用)

        depends_on            # 定义容器启动顺序 (此选项解决了容器之间的依赖关系， 此选项在 v3 版本中 使用 swarm 部署时将忽略该选项)
            示例：
                docker-compose up 以依赖顺序启动服务，下面例子中 redis 和 db 服务在 web 启动前启动
                默认情况下使用 docker-compose up web 这样的方式启动 web 服务时，也会启动 redis 和 db 两个服务，因为在配置文件中定义了依赖关系
                version: '3'
                services:
                    web:
                        build: .
                        depends_on:
                            - db      
                            - redis  
                    redis:
                        image: redis
                    db:
                        image: postgres                             

        dns                   # 设置 DNS 地址(等同于 docker run --dns 的作用)

        dns_search            # 设置 DNS 搜索域(等同于 docker run --dns-search 的作用)

        tmpfs                 # v2 版本以上, 挂载目录到容器中, 作为容器的临时文件系统(等同于 docker run --tmpfs 的作用, 在使用 swarm 部署时将忽略该选项)

        entrypoint            # 覆盖容器的默认 entrypoint 指令 (等同于 docker run --entrypoint 的作用)

        env_file              # 从指定文件中读取变量设置为容器中的环境变量, 可以是单个值或者一个文件列表, 如果多个文件中的变量重名则后面的变量覆盖前面的变量, environment 的值覆盖 env_file 的值
            文件格式：
                RACK_ENV=development 

        environment           # 设置环境变量， environment 的值可以覆盖 env_file 的值 (等同于 docker run --env 的作用)

        expose                # 暴露端口, 但是不能和宿主机建立映射关系, 类似于 Dockerfile 的 EXPOSE 指令

        external_links        # 连接不在 docker-compose.yml 中定义的容器或者不在 compose 管理的容器(docker run 启动的容器, 在 v3 版本中使用 swarm 部署时将忽略该选项)

        extra_hosts           # 添加 host 记录到容器中的 /etc/hosts 中 (等同于 docker run --add-host 的作用)

        healthcheck           # v2.1 以上版本, 定义容器健康状态检查, 类似于 Dockerfile 的 HEALTHCHECK 指令
            test                  # 检查容器检查状态的命令, 该选项必须是一个字符串或者列表, 第一项必须是 NONE, CMD 或 CMD-SHELL, 如果其是一个字符串则相当于 CMD-SHELL 加该字符串
                NONE                  # 禁用容器的健康状态检测
                CMD                   # test: ["CMD", "curl", "-f", "http://localhost"]
                CMD-SHELL             # test: ["CMD-SHELL", "curl -f http://localhost || exit 1"] 或者　test: curl -f https://localhost || exit 1
            interval: 1m30s       # 每次检查之间的间隔时间
            timeout: 10s          # 运行命令的超时时间
            retries: 3            # 重试次数
            start_period: 40s     # v3.4 以上新增的选项, 定义容器启动时间间隔
            disable: true         # true 或 false, 表示是否禁用健康状态检测和　test: NONE 相同

        image                 # 指定 docker 镜像, 可以是远程仓库镜像、本地镜像

        init                  # v3.7 中新增的参数, true 或 false 表示是否在容器中运行一个 init, 它接收信号并传递给进程

        isolation             # 隔离容器技术, 在 Linux 中仅支持 default 值

        labels                # 使用 Docker 标签将元数据添加到容器, 与 Dockerfile 中的 LABELS 类似

        links                 # 链接到其它服务中的容器, 该选项是 docker 历史遗留的选项, 目前已被用户自定义网络名称空间取代, 最终有可能被废弃 (在使用 swarm 部署时将忽略该选项)

        logging               # 设置容器日志服务
            driver                # 指定日志记录驱动程序, 默认 json-file (等同于 docker run --log-driver 的作用)
            options               # 指定日志的相关参数 (等同于 docker run --log-opt 的作用)
                max-size              # 设置单个日志文件的大小, 当到达这个值后会进行日志滚动操作
                max-file              # 日志文件保留的数量

        network_mode          # 指定网络模式 (等同于 docker run --net 的作用, 在使用 swarm 部署时将忽略该选项)         

        networks              # 将容器加入指定网络 (等同于 docker network connect 的作用), networks 可以位于 compose 文件顶级键和 services 键的二级键
            aliases               # 同一网络上的容器可以使用服务名称或别名连接到其中一个服务的容器
            ipv4_address      # IP V4 格式
            ipv6_address      # IP V6 格式

            示例:
                version: '3.7'
                services: 
                    test: 
                        image: nginx:1.14-alpine
                        container_name: mynginx
                        command: ifconfig
                        networks: 
                            app_net:                                # 调用下面 networks 定义的 app_net 网络
                            ipv4_address: 172.16.238.10
                networks:
                    app_net:
                        driver: bridge
                        ipam:
                            driver: default
                            config:
                                - subnet: 172.16.238.0/24

        pid: 'host'           # 共享宿主机的 进程空间(PID)

        ports                 # 建立宿主机和容器之间的端口映射关系, ports 支持两种语法格式
            SHORT 语法格式示例:
                - "3000"                            # 暴露容器的 3000 端口, 宿主机的端口由 docker 随机映射一个没有被占用的端口
                - "3000-3005"                       # 暴露容器的 3000 到 3005 端口, 宿主机的端口由 docker 随机映射没有被占用的端口
                - "8000:8000"                       # 容器的 8000 端口和宿主机的 8000 端口建立映射关系
                - "9090-9091:8080-8081"
                - "127.0.0.1:8001:8001"             # 指定映射宿主机的指定地址的
                - "127.0.0.1:5000-5010:5000-5010"   
                - "6060:6060/udp"                   # 指定协议

            LONG 语法格式示例:(v3.2 新增的语法格式)
                ports:
                    - target: 80                    # 容器端口
                      published: 8080               # 宿主机端口
                      protocol: tcp                 # 协议类型
                      mode: host                    # host 在每个节点上发布主机端口,  ingress 对于群模式端口进行负载均衡

        secrets               # 不知道怎么用

        security_opt          # 为每个容器覆盖默认的标签 (在使用 swarm 部署时将忽略该选项)

        stop_grace_period     # 指定在发送了 SIGTERM 信号之后, 容器等待多少秒之后退出(默认 10s)

        stop_signal           # 指定停止容器发送的信号 (默认为 SIGTERM 相当于 kill PID; SIGKILL 相当于 kill -9 PID; 在使用 swarm 部署时将忽略该选项)

        sysctls               # 设置容器中的内核参数 (在使用 swarm 部署时将忽略该选项)

        ulimits               # 设置容器的 limit

        userns_mode           # 如果Docker守护程序配置了用户名称空间, 则禁用此服务的用户名称空间 (在使用 swarm 部署时将忽略该选项)

        volumes               # 定义容器和宿主机的卷映射关系, 其和 networks 一样可以位于 services 键的二级键和 compose 顶级键, 如果需要跨服务间使用则在顶级键定义, 在 services 中引用
            SHORT 语法格式示例:
                volumes:
                    - /var/lib/mysql                # 映射容器内的 /var/lib/mysql 到宿主机的一个随机目录中
                    - /opt/data:/var/lib/mysql      # 映射容器内的 /var/lib/mysql 到宿主机的 /opt/data
                    - ./cache:/tmp/cache            # 映射容器内的 /var/lib/mysql 到宿主机 compose 文件所在的位置
                    - ~/configs:/etc/configs/:ro    # 映射容器宿主机的目录到容器中去, 权限只读
                    - datavolume:/var/lib/mysql     # datavolume 为 volumes 顶级键定义的目录, 在此处直接调用

            LONG 语法格式示例:(v3.2 新增的语法格式)
                version: "3.2"
                services:
                    web:
                        image: nginx:alpine
                        ports:
                            - "80:80"
                        volumes:
                            - type: volume                  # mount 的类型, 必须是 bind、volume 或 tmpfs
                                source: mydata              # 宿主机目录
                                target: /data               # 容器目录
                                volume:                     # 配置额外的选项, 其 key 必须和 type 的值相同
                                    nocopy: true                # volume 额外的选项, 在创建卷时禁用从容器复制数据
                            - type: bind                    # volume 模式只指定容器路径即可, 宿主机路径随机生成; bind 需要指定容器和数据机的映射路径
                                source: ./static
                                target: /opt/app/static
                                read_only: true             # 设置文件系统为只读文件系统
                volumes:
                    mydata:                                 # 定义在 volume, 可在所有服务中调用

        restart               # 定义容器重启策略(在使用 swarm 部署时将忽略该选项, 在 swarm 使用 restart_policy 代替 restart)
            no                    # 禁止自动重启容器(默认)
            always                # 无论如何容器都会重启
            on-failure            # 当出现 on-failure 报错时, 容器重新启动

        其他选项：
            domainname, hostname, ipc, mac_address, privileged, read_only, shm_size, stdin_open, tty, user, working_dir
            上面这些选项都只接受单个值和 docker run 的对应参数类似

        对于值为时间的可接受的值：
            2.5s
            10s
            1m30s
            2h32m
            5h34m56s
            时间单位: us, ms, s, m， h
        对于值为大小的可接受的值：
            2b
            1024kb
            2048k
            300m
            1gb
            单位: b, k, m, g 或者 kb, mb, gb
    networks          # 定义 networks 信息
        driver                # 指定网络模式, 大多数情况下, 它 bridge 于单个主机和 overlay Swarm 上
            bridge                # Docker 默认使用 bridge 连接单个主机上的网络
            overlay               # overlay 驱动程序创建一个跨多个节点命名的网络
            host                  # 共享主机网络名称空间(等同于 docker run --net=host)
            none                  # 等同于 docker run --net=none
        driver_opts           # v3.2以上版本, 传递给驱动程序的参数, 这些参数取决于驱动程序
        attachable            # driver 为 overlay 时使用, 如果设置为 true 则除了服务之外，独立容器也可以附加到该网络; 如果独立容器连接到该网络，则它可以与其他 Docker 守护进程连接到的该网络的服务和独立容器进行通信
        ipam                  # 自定义 IPAM 配置. 这是一个具有多个属性的对象, 每个属性都是可选的
            driver                # IPAM 驱动程序, bridge 或者 default
            config                # 配置项
                subnet                # CIDR格式的子网，表示该网络的网段
        external              # 外部网络, 如果设置为 true 则 docker-compose up 不会尝试创建它, 如果它不存在则引发错误
        name                  # v3.5 以上版本, 为此网络设置名称
文件格式示例：
    version: "3"
    services:
      redis:
        image: redis:alpine
        ports:
          - "6379"
        networks:
          - frontend
        deploy:
          replicas: 2
          update_config:
            parallelism: 2
            delay: 10s
          restart_policy:
            condition: on-failure
      db:
        image: postgres:9.4
        volumes:
          - db-data:/var/lib/postgresql/data
        networks:
          - backend
        deploy:
          placement:
            constraints: [node.role == manager]
```
### docker compose常用命令  
#### docker-compose up  
用于部署一个compose应用，默认情况下该命令会读取名为docker-compose.yml或者dokcer-compose.yaml的文件。当然用户也可以使用-f指定其他文件名，通常情况下使用-d参数应用后台启动  
#### docker-compose stop  
停止compose应用相关的所有容器，但不会删除它们。被停止的应用可以很容易地使用docker-compose restart命令重新启动  
#### docker-compose rm  
用于删除已停止地compose应用，会删除容器和网络，但是不会删除卷和镜像  
#### docker-compose restart  
重启已停止的compose应用，如果用户在停止以后对其进行了变更，那么变更的内容不会反映在重启的应用中，这时需要重新部署应用使变更生效  
#### docker-compose ps  
用于列出compose应用中的各个容器，输出的内容包括当前状态、容器运行命令以及网络端口  
#### docker-compose down  
停止并删除运行中的compose应用，会删除容器和网络，但是不会删除卷和镜像  
## docker网络  
### 基础理论  
在顶层设计中，docker网络架构由3个主要部分组成：CNM、Libnetwork和驱动  
CNM是设计的标准，在CNM中，规定了docker网络架构的基础组成要素  
Libnetwork是CNM的具体实现，并且被docker采用，Libnetwork通过Go语言编写，并实现了CNM中列举的核心组件  
驱动通过实现特定网络拓扑的方式来拓展该模型的能力  
下图展示了顶层设计中的每个部分是如何组装在一起的  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/27/1590544671972-1590544672011.png)  
### CNM  
docker网络架构的设计规范是CNM  
抽象的来讲，CNM定义了3个基本要素：沙盒(sandbox)、终端(Endpoint)和网络(Network)  
沙盒是一个独立的网络栈。其中包括以太网接口、端口、路由表以及DNS配置  
终端就是虚拟网络接口。就像普通网络一样，终端主要职责是创建连接。在CNM中，终端负责将沙盒连接到网络  
网络是802.1d网桥的软件实现。因此，网络就是需要交互的终端集合，并且终端直接相互独立  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/27/1590544948389-1590544948390.png)  
下图展示了CNM组件是如何与容器进行关联的----将沙盒放置在容器的内部，为容器提供网络连接  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/27/1590545028154-1590545028155.png)  
容器A只有一个接口(终端)并且连接到了网络A。容器B由两个接口(终端)并且分别接入了网络A和网络B。容器A与容器B之间是相互可以通信的，因为都接入了网络A。但是，如果没有三层路由的支持，容器B的两个终端之间是不能相互通信的。  
终端与常见的网络适配器类似，这意味着终端只能接入某一个网络，因此如果容器需要接入到多个网络，就需要多个终端  

### Libnetwork  
Libnetwork实现了CNM中定义的全部三个组件，此外还实现了本地服务发现(service discovery)，基于Ingress的容器负载均衡，以及网络控制层和管理层功能  
### 驱动  
如果说Libnetwork实现了控制层和管理层功能，那么驱动就是负责数据层的实现。比如，网络连通性和隔离性是由驱动来处理的，驱动层实际创建网络对象也是如此，其关系如下图  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/27/1590546603382-1590546603389.png)  
在linux上，包括Bridge、Overlay以及Macvlan  
## docker单机桥接网络详解  
单机桥接意味着只能在单个docker主机上运行，并且只能与所在docker主机上的容器进行连接，桥接意味着这是802.1d桥接的一种实现(二层交换机)  
每个docker主机上都有一个默认的单机桥接网路，linux上网络名称为birdge，除非通过命令行创建容器时指定参数--network，否则默认情况下，新创建的容器都会连接到该网络  
在linux主机上，docke网络由bridge驱动创建  
在linux docker主机上，默认的bridge网络被映射到内核中为"docker0"的linux网桥，如下图所示  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/27/1590548203632-1590548203634.png)  
"bridge"网络在主机内核中映射到名为"docker0"的linux网桥，该网桥可以通过主机以太网接口的端口进行反向关联  
提示：linux默认的bridge网络是不支持通过docker dns服务进行域名解析的，自定义桥接网络可以支持  
到目前为止，前面提到的单机桥接网络中的容器只能与位于同一网络中容器进行通信，其实还可以通过端口映射的方式来绕开这个限制。端口映射允许将某个容器端口映射到docker主机端口上，对于配置中指定的docker主机端口，任何发送到该端口的流量都会被转发到容器中
## docker网络模式  
### birdge模式  
当docker进程启动的时候，会在主机上创建一个docker0的虚拟网桥，此主机上的docke容器会连接到这个虚拟网桥上。虚拟网桥的工作方式就和物理交换机类似，这样主机上的所有容器都会连接到一个二层网络中。从docker0子网中分配一个ip给容器使用，并设置docker0的ip地址为容器的默认网关。在主机上创建一对虚拟网卡veth pair设备，docker将veth pair设备的一端放在新创建的容器中，另一端放在主机中  
bridge模式是docker默认网络模式，不写`-net`参数，就是bridge模式。使用`docker run -p`时，docker实际是在iptables中做了DNAT规则，实现端口转发 
`-net=bridge`指定，默认使用此模式   
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/29/1590734943836-1590734943838.png)  
### host模式  
如果启动docker的时候是使用host模式，那么这个容器将不会获得一个独立的network workspace，而是和宿主机共用一个network namespace。容器将不会虚拟出自己的网卡，配置自己的ip等，而是使用宿主机的ip和端口。但在其他方面，还是和宿主机隔离的  
`-net=host`指定  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/29/1590735384868-1590735384871.png)  
### container模式  
这个模式指定创建的容器和已经存在的一个容器共享一个network namespace，而不是和宿主机共享。新创建的容器不会创建自己的ip，网卡。同样两个容器除了网络方面，其他的都是相互隔离的。两个容器可以通过lo网卡设备通信  
`net=container:NAME or ID`指定  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/29/1590736014453-1590736014455.png)  
### none模式  
使用none模式，docker拥有自己的network namespace，但是并不为docker进行任何网络配置。也就是说，这个docker容器没有网卡，ip，路由信息等，需要自己来手动添加  
`-net=none`指定  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/29/1590736238540-1590736238548.png)  
## docker macvaln  
docker内置的macvaln驱动能够通过为容器提供mac和ip地址，让容器能够成为物理网络的"一等公民"  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/27/1590549942308-1590549942317.png)  
macvlan的优点是性能优异，无须端口映射或者额外桥接，可以直接通过主机接口(或者子接口)访问容器接口，但是macvaln的缺点是需要将主机的网络(NIC)设置为混杂模式，这在大部分公有云平台上是不允许的  
可以通过`docker container logs`命令查看单独的容器日志，通过`docker service logs`可以查看swarm服务日志  
### docker network网络子命令  
主要的docker网络子命令如下表所示  
|子命令|说明|
|-|-|
|`docker network connect`|将容器连接到网络|
|`docker network create`|创建新的docker网络，可以使用-d参数指定网络类型|
|`docker network disconnect`|断开docker的网络连接|
|`docker network inspect`|提供docker网络的详细配置|
|`docker network ls`|列出运行于本地的docker主机上的全部网络|
|`docker network prume`|删除主机上未被使用的网络|
|`docker network rm`|删除主机上指定的网络|
### docker卷与持久化  
docker对持久化的数据和非持久化的数据都有支持  
非持久化存储自动创建，从属于容器，生命周期与容器相同。这意味着删除容器就会删除全部的非持久化数据  
持久化的数据则需要存储在卷上。卷与容器时解耦的，从而独立地创建并管理卷，并且卷的生命周期与容器并不相同。用户删除了一个关联了卷的容器，也不会删除容器中的内容  
### 容器与持久化数据  
在容器中持久化数据的方式推荐使用卷  
总体来说，用户创建卷，然后创建容器，接着将卷挂载到容器上。卷会挂载都容器的某个目录之下，任何写到改目录的内容都会写到卷中。即使用户删除了容器，卷中的内容也会保留下来  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/29/1590740607945-1590740607948.png)  
`docker volume crate volume-name`使用这个命令可以创建一个卷。默认情况下，docker创建新卷时使用内置的local驱动，本地卷也只能被所在节点的容器使用。使用-d参数可以指定不同的驱动 
docker 卷可以如下使用：  
```shell
docker volume create myapp
docker run --rm -it --mount 'type=volume,src=myapp,dst=/var,volume-driver=local' 5e8b97a2a082 
```
### docker volume子命令  
|命令|说明|
|-|-|
|`docker volume create`|创建新的卷，默认情况下使用local驱动|
|`docker volume ls`|列出本地主机上的所有卷|
|`docker volume inspect`|用于详细查看卷的具体信息|
|`docker volume prume`|删除未被使用的卷|
|`docker volume rm`|删除指定的卷|
## docker swarm  
docker swarm是docker官方提供的一款集群管理工具，其主要作用是把若干台docker主机抽象成为一个整体，并且通过一个入口统一管理这个docker主机上的各种docker资源  
swarm和kubernetes比较类似，但是更加轻，具有的功能也比较少一些  
docker swarm包含两个方面，一个企业级的docker安全集群，以及一个微服务应用编排引擎  
### docker swarm初步介绍  
从集群的角度说，一个swarm由一个或多个docker节点组成，唯一要求的前提就是要求就是所有的节点通过可靠的网络相连  
节点会被配置为管理节点(manager)或者工作节点(worker).管理节点负责集群控制面(control plane)，进行诸如监控集群状态、分发任务到管理工作节点等操作。工作节点接收来自管理节点的任务并执行    
swarm的配置和状态信息保存在一套位于所有管理节点的分布式etcd数据库中，该数据库运行于内存之中，并保持数据的最新状态。作为swarm的一部分被安装，无需管理  
swarm使用TLS进行通信加密、节点认证和角色授权  
swarm中的最小调度单元是服务，它是随swarm引入的，在api中是一个新的对象元素，它基于容器封装了一些高级特性，是一个更高层次的概念。当容器被封装在一个服务中时，我们称之为一个任务或一个副本，服务中增加了诸如扩缩容、滚动升级以及简单回滚等特性  
从概括性的视角来看swarm   
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/06/01/1591013048303-1591013048348.png)  
### docker swarm集群搭建  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/06/02/1591070154398-1591070154436.png)  
每个节点都需要安装docker，并且能够与swarm的其他节点通信  
### 初始化一个全新的swarm  
不包含在任何swarm中的docker节点，称为运行在单引擎(single-engine)模式，一旦被加入了swarm集群，则切换为swarm模式  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/06/02/1591070293495-1591070293496.png)  
在单引擎模式下的docker主机上运行`docker swarm init`会将其切换到swarm模式下，并创建一个新的swarm，将自身设置为swarm的第一个管理节点  
#### 初始化一个新的swarm  
`docker swarm init --advertise-addr 10.0.0.1:2337 --listen-addr 10.0.0.1:2337`  
`docker swarm init`会通知docker来初始化一个新的swarm，并将自身设置为第一个管理节点，同时也会使该节点开启swarm模式  
`--advertise-addr<format: <ip|interface>[:port]>`指定其他节点用来连接到当前管理节点的IP和端口，这一属性是可选的,当节点有多个IP时，可以指定使用哪个IP。此外，还可以用于指定一个节点上没有的远程IP，比如负载均衡的IP  
`--listen-addr<format: <ip|interface>[:port]>`指定用于承载swarm流量的IP和端口，其设置通常与`--advertise-addr`相匹配，有多个节点的时候可以指定使用哪个IP  
`docker swarm join-token worker`命令用来获取添加新的工作节点到swarm的命令和token  
`docker swarm join-token manager`命令用来获取新的管理检点到swarm的命令和token  
```shell
$ docker swarm join-token worker  
To add a worker to this swarm,run the following command:
dokcer swarm join --token SWMTKN-1-0uahebax...c87tu8dx2c  10.0.0.1:2377

$ docker swarm join-token manager  
To add a manager to this swarm,run the following command:
dokcer swarm join --token SWMTKN-1-0uahebax...ue4hv6ps3p  10.0.0.1:2377
```
请注意，工作节点和管理节点的接入命令使用的Token是不一样的，因为一个节点是作为工作节点还是管理节点接入完全取决于Token  
#### 接入工作节点  
`docker swarm join --token SWMTKN-1-0uahebax...c87tu8dx2c 10.0.0.1:2377 --advertise-addr 10.0.0.4:2377 --listen-addr 10.0.0.4:2377`  
`--advertise-addr`和`--listen-addr`的属性是可选的，在网络配置方面应该尽量明确的指明  
## swarm 管理器的高可用性  
swarm的管理节点内置有对HA的支持，这意味着，即使一个或多个节点发生故障，剩余的管理节点也会继续保证swarm的运转  
通常处于活动状态的管理节点被称为"主节点"，而主节点也是唯一一个会对swarm发送控制命令的节点，也就是说，只有主节点才会变更配置，或者发送任务到工作节点；如果有一个备用管理节点接收到了swarm的命令，则它会将命令转发给主节点  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/06/02/1591079557481-1591079557482.png)  
关于HA，有以下两条最佳实践  
1.部署奇数个管理节点  
2.不要部署太多管理节点(3个或5个)  
## 内置swarm安全机制  
### 锁定swarm  
尽管内置有多种的原生安全机制，重启一个旧的管理节点或进行备份恢复仍有可能对集群造成影响  
一个旧的管理节点重新接入swram会自动解锁并获得Raft数据库中长时间队列的访问权，这会带来安全隐患  
进行备份恢复可能会抹掉最新的swarm的配置  
为了规避以上问题，docker提供了自动解锁机制来锁定swarm，这会强制要求重启的管理节点在提供一个集群解锁密码以后才有权重新接入集群  
可以在执行`docker swarm init`的命令时指定参数`--autolock`参数可以直接启动锁，也可以使用`docker swarm
 update `命令来启用锁  
 ```shell
$ docker swarm update --autolock=true
Swarm updated
To unlock a swarm manager after it restarts,run the `docker swarm unlock`command and provide the following key:  
SWMKEY-1-5+ICW2kRxPxZrVyBDWzBkzZdSd0Yc7Cl2o4Uuf9NPU4  
Please remember to store this key in a password manager, since without
it you will not be able to restart the manager.
 ```
执行`docker swarm unlock`命令来为重启的管理节点解锁swarm，该命令需要在重启的节点上执行，并提供解锁码  
```shell
$ docker swarm unlock  
Please enter unlock key:<enter your key>
```
该节点被允许重新接入swarm
## swarm服务部署  
使用服务仍然能够配置大多数熟悉的容器配置，比如容器端口等，此外还增加了额外的特性，比如可以声名应用服务的期望状态，将其告知docker后，docker会负责进行服务的部署和管理  
```shell
docker service create --name web-fe -p 8080:8080 --replicas 5 nigelpoulton/pluralsight-docker-ci
```
需要注意的是，该命令与熟悉的docker container run的命令的许多参数是相同的  
通过上面的命令，可以看出，`docker service create`命令告知docker正在声明一个新服务，并传递--name参数将其命名为web-fe,将每个节点上的8080端口映射到服务副本内部的8080端口，接下来，使用--replicas参数告知docker该服务总是有5个副本  
所有的服务都会被swarm持续监控，swarm会在后台进行轮询检查，来持续比较服务的实际状态和期望的状态是否一致。如果一致，则无需任何额外的操作，如果不一致，swarm会使其一致，swarm会一致确保实际状态能够满足期望状态的要求  
例如，运行web-fe副本的某个工作节点宕机了，则web-fe的状态会从5个副本降为4个副本，从而不能满足期望状态的要求，docker会启动一个新的web-fe副本来使实际状态与期望状态保持一致  
### 副本服务VS全局服务  
服务的默认复制模式(Replication Mode)是副本模式(replicated)  
这种模式会期望部署数量的副本，并尽可能均匀地将各个副本分布在整个集群  
另一种模式是全局模式(global)，在这种模式下，每个节点上仅运行一个副本，可以通过`docker service create`命令传递参数--model global参数来部署一个全局服务  
### 服务地扩缩容  
`docker service scale web-fe=10`该命令会将服务副本数由5个增加到10个，后台会将服务的期望状态从5个增加到10个  
### 滚动升级  
演示一下如何服务滚动升级，在此之前先创建一个overlay网络  
`docker network create -d overlay uber-net`该命令会创建一个overlay网络，该网络是一个二层网络，容器可以接入该网络，并且所有接入的容器均可互相通信  
即使这些容器所在主机的网络接入的是不同的底层网络，也是互通的  
```shell
$ docker service create --name uber-svc \
--network uber-net \
-p 80:80 --replicas 12 \
nigelpoulton/tu-demo:v1
```
上面的命令首先将服务命名为uber-svc，并用--network参数声明所有的副本都连接到uber-net网络，然后再整个swarm中将80端口暴露出来，并将其映射到12个容器副本的端口，最后声明所有的副本都是基于nigelpoulton/tu-demo:v1镜像  
默认的模式，实在swarm中的所有节点开放端口，称为入站模式(Ingress Mode),此外还有主机模式(Host Mode)，即仅在运行有容器副本的节点开放端口  
以主机模式开放服务端口，需要较长的格式声明语法  
```shell
docker service create --name uber-svc \
--network uber-net \
--publish published=80,target=80,mode=host \
--replicas 12 \
nigelpoulton/tu-demo:v1
```
此外假设，本次升级任务在将新镜像更新到swarm中时采用一种阶段性的方式，每次更新两个副本，并且中间间隔20s  
```shell
docker service update \
--image nigelpoulton/tu-demo:v2 \
--update-parallelism 2 \
--udpate-delay 20s uber-svc
```
分析以上的命令，指定了tag为v2的新镜像，接下来使用--update-parallelism 和--update-delay 参数声明每次使用新镜像更新两个副本，期间有20s的延迟  
### docker swarm服务日志及相关配置  
docker swarm服务的日志可以通过执行`docker swarm logs`命令来查看，然而并非所有的日志驱动都支持该命令  
docker节点的默认配置是服务使用json-file日志驱动，其他的驱动还有journald、syslog、splunk和gelf  
json-file和journald是比较容易配置的，二者都可以使用`docker srevice logs`命令  
命令的格式为`docker service logs <service-name>`  
如果使用第三方日志驱动，就需要用相应的日志平台的原生工具来查看日志  
如下是在daemon.json配置文件中定义使用syslog作为日志驱动  
```shell
{
    "log-drive":"syslog"
}
```
通过在执行`docker service create`命令时传入--logdriver 和 --log-opts参数可以强制某服务使用一个不同的日志驱动，这会覆盖daemon.json中的配置  
服务日志能够正常工作的前提是容器内的应用程序运行于PID为1的进程，并且将日志发送给STDOUT，错误信息发送给STDERR，日志驱动会将这些日志转发到其配置的指定的位置  
对于查看日志命令，可以使用--follow进行跟踪，使用--tail 显示最近的日志，并使用--details获取额外的信息  
### docker swarm常用命令  
|命令|说明|
|-|-|
|docker swarm init|用于创建一个新的swarm，执行该命令的节点会称为第一个管理节点，并且会自动切换到swarm模式|
|docker swarm join-token|用于查询加入管理节点和工作节点到现有swarm时所使用的命令和Token|
|docker node ls|用于列出swarm中的所有节点及相关信息，包括哪些是管理节点，哪些是主管理节点|
|docker service create|用于创建一个新的服务|
|docker serivice ls|用于列出swarm中运行的服务，以及服务状态、服务副本等基本信息|
|docker service ps <service>|该命令会给出更多关于某个服务副本的信息|
|docker sercice inspect|用于获取关于服务的详尽信息，附件--pretty参数可限制仅显示重要信息|
|docker service scale|用于对服务副本个数增加或减少|
|docker service update|用于对运行中的服务的属性进行变更|
|docker service logs|用于查看服务的日志|
|docker service rm|用于从swarm中删除某服务，该命令会在不做确认的情况下删除服务的所有副本，所以使用时应保持警惕|
## docker swarm服务发布模式  
通过Ingress模式发布的服务，可以保证从swarm集群内任一节点(即使没有运行服务的副本)都能访问该服务，以host模式发布的服务只能通过运行服务副本的节点来访问  
::: hljs-center

![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/06/03/1591175728762-1591175728856.png) 

:::  
Ingress模式是默认方式，这意味着任何时候通过-p或者--publish发布服务的时候，默认都是Ingress模式；如果需要以Host模式发布服务，则都读者需要使用--publish参数的完整格式，并添加mode=host  
```shell
docker service create -d --name svc1 \
--publish published=5000,target=80,mode=host \
nginx
```
published=5000表示服务通过端口5000提供外部服务
target=80表示发送到published端口5000的请求，会映射到服务副本的80端口只上
mode=host表示只有外部请求发送到运行了服务副本的节点才可以访问服务  
::: hljs-center
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/06/03/1591176253807-1591176253809.png)
:::  
按上述方式发布的swarm服务(--publish published=5000,target=80)会在Ingress网络的5000端口进行发布，因为swarm全部节点都接入了Ingress网络，所以这个端口被发布到了swarm范围内  
集群确保达到Ingress网络中任意节点的5000端口的流量，都会被路由到80端口的"svc1"服务  
::: hljs-center

![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/06/03/1591176633454-1591176633456.png)  

:::

如果服务只有一个副本，所有访问Ingress网络5000端口的流量都需要路由到这个副本上；如果存在多个运行的副本，流量会平均到每个服务上   







 


































