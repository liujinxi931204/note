# docker学习  
当安装docker的时候会涉及到两个主要的组件：docker客户端和docker daemon(也称为服务端或者docker引擎)，其中docker daemon实现了docker引擎的API。在linux环境下安装的时候，客户端和daemon的通信时通过IPC/Unix socket实现的（/var/run/docker.sock）
## docker引擎
