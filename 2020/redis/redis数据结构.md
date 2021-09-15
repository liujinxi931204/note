## 常用的数据类型  

redis中常用的数据类型有：字符串类型，列表类型、哈希表类型、集合类型和有序集合类型  

## redis核心对象  

在redis中有一个核心对象`redisObject`，是用来表示所有的key和value的，用`redisObject`结构体来表示`string、list、hash、set和zset`五种数据类型  

`redisObject`的定义位于`redis.h`  

```c
typedef struct redisObject {
    unsigned type:4; // 4 bit
    unsigned encoding:4; // 4 bit
    unsigned lru:LRU_BITS; // 3 个字节
    int refcount; // 4 个字节
    void *ptr; // 8 个字节
} robj;
```

![redisObject数据结构](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/redisObject%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.png)      

在`redisObject`中，`type`表示属于哪种数据类型，比如`string`类型、`hash`类型等。`encoding`表示该数据类型底层的存储结构，比如`zset`类型的底层存储结构可能是`ziplist`，也可能是`skiplist`。`ptr`是数据指针，就是指向底层存储结构的指针

## 字符串  

字符串是最常用的数据类型，`set`和`get`命令就是字符串的操作命令  

### 简单动态字符串（Simple Dynamic String，SDS）  

`redis`自己构建了一种名为简单动态字符串的抽象类型，并将SDS用作`redis`默认字符串的表示。SDS的结构如下  

```c
typedef char *sds;

struct __attribute__ ((__packed__)) sdshdr5 { // 对应的字符串长度小于 1<<5
    unsigned char flags;
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 { // 对应的字符串长度小于 1<<8
    uint8_t len; /* 已使用长度，1 字节存储 */
    uint8_t alloc; /* 总长度 */
    unsigned char flags; 
    char buf[]; // 真正存储字符串的数据空间
};
struct __attribute__ ((__packed__)) sdshdr16 { // 对应的字符串长度小于 1<<16
    uint16_t len; /* 已使用长度，2 字节存储 */
    uint16_t alloc; 
    unsigned char flags; 
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 { // 对应的字符串长度小于 1<<32
    uint32_t len; /* 已使用长度，4 字节存储 */
    uint32_t alloc; 
    unsigned char flags; 
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 { // 对应的字符串长度小于 1<<64
    uint64_t len; /* 已使用长度，8 字节存储 */
    uint64_t alloc; 
    unsigned char flags; 
    char buf[];
};
```

SDS的表示如下图  

![简单动态字符串](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E7%AE%80%E5%8D%95%E5%8A%A8%E6%80%81%E5%AD%97%E7%AC%A6%E4%B8%B2.png)  

### 具体实现  

如果一个字符串对象保存的是整数值，并且这个整数值可以用`long`类型来表示，那么字符串对象会将整个数值保存在`redisObject`的`ptr`属性里（将`void*`转换成`long`），并将字符串对象的编码设置为`int`。 此时具体实现可以参考下图  

![redis字符串int编码](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/redis%E5%AD%97%E7%AC%A6%E4%B8%B2int%E7%BC%96%E7%A0%81.png)    

如果字符串对象保存的是一个字符串值，并且这个字符串值的长度大于44字节的时候，那么字符串对象将使用一个简单动态字符串（SDS）来表示这个字符串值，并且将对象的编码设置为`raw`  

如果字符串对象保存的是一个字符串值，并且这个字符串值的长度小于等于44字节的时候，那么字符串对象将使用一个简单动态字符串（SDS）来表示这个字符串值，并且将这个对象的编码设置为`embstr`

那`raw`的编码方式和`embstr`的编码方式有什么区别呢？`raw`会调用两次内存分配函数创建`redisObject`结构和`sdshdr`结构，而`embstr`编码方式则通过调用一次内存分配函数分配一块来连续的空间，空间内一次包含了`redisObject`和`sdshdr`两个结构  







