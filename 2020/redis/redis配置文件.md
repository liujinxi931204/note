在Redis中，根目录中有一个Redis配置文件(redis.conf)，可以通过Redis CONFIG命令获取和设置所有的redis配置  
句法：  
`CONFIG GET CONFIG_SETTING_NAME`  
例如：  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/09/07/1599464279736-1599464279819.png)  
要获取所有的配置设置，请使用"*"代替"CONFIG_SETTING_NAME"  
即`CONFIG GET *`  
编辑配置  
1. 可以直接编辑redis.conf
2. 通过`CONFIG SET CONFIG_SETTING_NAME NEW_CONFIG_VALUE`的命令设置  
例如：  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/09/07/1599464770375-1599464770384.png)  
## 常用配置参数说明  
#### redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程  
`daemonize no`  
#### 当redis以守护进程方式运行时，redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定  
`pidfile /var/run/redis.pid`  
#### 指定redis监听端口，默认端口为6379，可以通过port指定  
`port 6379`  
#### 绑定主机地址，通过bind绑定主机地址  
`bind 127.0.0.1`  
#### 当客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能  
`timeout 300`  
#### 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose  
`loglevel verbose`  
#### 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里而又配置日志记录方式为标准输出，则日志将会发送到/dev/null  
`logfile stdout`  
#### 设置数据库的数量，默认数据为16，可以使用SELECT <db_id>命令连接指定id的数据库  
`databases 16`  
#### 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合  
`save <seconds> <changes>`  
redis默认配置文件中提供了三个条件：  
```shell
save 900 1
save 300 10
save 60 10000
```  
分别表示900秒内有1个更改，300秒内有10个更改以及60秒内有10000个更改就将数据同步给数据文件  
#### 指定存储至本地数据库时是否压缩数据，默认为yes，redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大  
`rdbcompression yes`  
#### 指定本地数据库文件名，默认值为dump.rdb  
`dbfilename dump.rdb`  
#### 指定本地数据库存放目录  
`dir ./`  
#### 设置当本机为salve服务时，设置master服务的ip地址及端口，在redis启动时，它会自动从master进行数据同步  
`salveof <masterip> <masterport>`  
#### 当master设置了密码保护时，slave服务连接master的密码  
`masterauth <master-password>`  
#### 设置redis连接密码，如果设置了连接密码，客户端在连接redis时需要通过AUTH <password>命令提供密码，默认关闭  
`requirepass foobared`  



  



 

  

  
