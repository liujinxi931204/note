在Redis中，根目录中有一个Redis配置文件(redis.conf)，可以通过Redis CONFIG命令获取和设置所有的redis配置  
句法：  
`CONFIG GET CONFIG_SETTING_NAME`  
例如：  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/07/1599464279736-1599464279819.png)  
要获取所有的配置设置，请使用"*"代替"CONFIG_SETTING_NAME"  
即`CONFIG GET *`  
编辑配置  
1. 可以直接编辑redis.conf
2. 通过`CONFIG SET CONFIG_SETTING_NAME NEW_CONFIG_VALUE`的命令设置  
例如：  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/07/1599464770375-1599464770384.png)  
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
#### 设置同一时间最大客户端连接数，默认无限制，redis可以同时打开的客户端连接数为redis进程可以打开的最大文件描述符数，如果设置maxclinets 0，表示不作限制。当客户端连接数达到限制时，redis会关闭新的连接并向客户端返回max number of clients reached错误信息  
`maxclients 128`  
#### 指定redis最大内存限制，redis在启动时会把数据加载到内存中，达到最大内存后，redis会尝试清除已到期或者即将到期的key，当此方法处理后，仍然达到最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。redis新的vm机制，会把key存放在内存，value存放在swap区  
`maxmemory <bytes>`  
#### 指定是否在每次更新操作后进行日志记录，redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中  
`appendonly no`  
#### 指定更新日志文件名，默认为appendfile.aof  
`appendfilename appendonly.aof`  
#### 指定更新日志条件，共有3个可选值。no：表示操作系统进行数据缓存同步到磁盘(块)；always：表示每次更新操作后手动调用fsync()将数据写到洗盘(慢，安全);everysec:表示每秒同步一次(折中，默认值)  
`appendfsync everysec`  
#### 指定是否启用虚拟内存机制，默认值为no。VM机制将数据分页存放，由redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中  
`vm-enable no`  
#### 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个redis实例共享  
`vm-swap-file /tmp/redis.swap`  
#### 将所有大于vm-max-memory的所有数据存入虚拟内存，无论vm-max-memory设置多小，所有的索引数据都是内存存储的(redis的索引数据就是key),也就是说，当vm-max-memory设置为0的时候，其实就是所有value都存在于磁盘，默认值为0  
`vm-max-memory 0`  
 #### redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page不能被多个对象共享，vm-page-size是要根据存储的数据大小来设定的。如果存储很多小对象，page大小最好设置为32或64byte；如果存储很大的对象，则可以使用更大的page；如果不确定，就是用默认值  
`vm-page-size 32`  
#### 设置swap文件中的page数量，由于页表(一种表示页面空闲或使用的bitmap)是存放在内存中的，在磁盘上每8个pages将会消耗1byte的内存  
`vm-pages 134217728`  
#### 设置访问swap文件的线程数，最好不要超过机器的核数；如果设置为0，那么多所有的swap文件的操作都是串行的，可能会造成比较长时间的延迟，默认值为4  
`vm-max-threads 4`  
#### 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认开启  
`glueoutputbuf yes`  
#### 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的hash算法  
```shell
hash-max-zipmap-entries 64
hash-max-zipmap-value 512
```
#### 只当是否激活充值hash，默认为开启  
`activerehashing yes`  
#### 指定包含其他的配置文件，可以在同一个主机上多个redis实例之间使用同一份配置为文件，而同时各个实例又拥有自己的特定配置文件  
`include /path/to/local.conf`  







 











