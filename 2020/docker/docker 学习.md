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



