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

```
