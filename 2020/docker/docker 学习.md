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

