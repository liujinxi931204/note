## 常用的数据类型  

redis中常用的数据类型有：字符串类型，列表类型、哈希表类型、集合类型和有序集合类型  

## redis核心对象  

在redis中有一个核心对象`redisObject`，是用来表示所有的key和value的，用redisObject结构体来表示string、list、hash、set和zset五种数据类型  

![redisObject数据结构](https://gitee.com/liujinxi931204/typoraImage/raw/master/redisObject%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.png)    

在redisObject中，type表示属于哪种数据类型，比如string类型、hash类型等。encoding表示该数据类型底层的存储结构，比如zset类型的底层存储结构可能是ziplist，也可能是skiplist。ptr是数据指针，就是指向底层存储结构的指针