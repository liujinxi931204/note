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
5. 修改配置  
```shell
vim /search/odin/redis/etc/redis.conf
# redis以守护进程的方式运行
# no表示不以守护进程的方式运行(会占用一个终端)
daemonize yes
# 客户端闲置多长时间后断开连接，默认为0关闭
```
