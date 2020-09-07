## 编译安装  
1. 下载源码到当前目录(这里是/search/odin/)  
`wget http://download.redis.io/releases/redis-5.0.8.tar.gz`  
2. 解压
`tar zxvf redis-5.0.8.tar.gz`  
3. 安装依赖  
`yum -y install gcc gcc-c++ kernel-devel`  
4. 安装  
```shell
cd redis-5.0.8
make PREFIX=/search/odin/redis install
mkdir -p /search/odin/redis/etc/
cp redis.conf /search/odin/redis/etc/
``` 
5. 修改配置(简单配置两个选项)  
```shell
vim /search/odin/redis/etc/redis.conf
# redis以守护进程的方式运行
# no表示不以守护进程的方式运行(会占用一个终端)
daemonize yes
# 客户端闲置多长时间后断开连接，默认为0关闭此功能
timeout 300
```
6. 启动redis  
`/search/odin/redis/bin/redis-server redis.conf`
7. 启动redis后，可以使用测试端程序redis-cli和redis服务交互  
`/search/odin/redis/bin/redis-cli`
8. 关闭redis  
``
