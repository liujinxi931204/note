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

那`raw`的编码方式和`embstr`的编码方式有什么区别呢？`raw`会调用两次内存分配函数创建`redisObject`结构和`sdshdr`结构，而`embstr`编码方式则通过调用一次内存分配函数分配一块来连续的空间，空间内一次包含了`redisObject`和`sdshdr`两个结构。此外，`embstr`编码的字符串对象实际上是只读的，`redis`没有为`embstr`编码的字符串对象编写任何相应的修改程序。因此，当需要对`embstr`编码的字符串对象执行任何修改命令时（例如`append`），程序会先将对象的编码从`embstr`转化为`raw`，然后再执行修改命令  

这两种编码方式如下图所示  

![redis字符串两种编码方式](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/redis%E5%AD%97%E7%AC%A6%E4%B8%B2%E4%B8%A4%E7%A7%8D%E7%BC%96%E7%A0%81%E6%96%B9%E5%BC%8F.png)  

## 列表  

列表类型是一个使用链表存储的有序结构，它的元素插入会按照先后顺序存储到链表结构中，因此它的元素操作（插入、删除）时间复杂度为`O(1)`，但是它的查询时间复杂度为`O(n)`，因此查询可能会比较慢  

列表对象的编码早期版本是`ziplist`和`linkedlist`，也就是元素数量少时使用`ziplist`，元素数量多时使用`linkedlist`。在`redis 3.2`以后改为使用`quicklist`来存储元素。`quicklist`是`ziplist`和`linkedlist`的结合体，它将`linkedlist`  按段切分，每一段使用`ziplist`来紧凑存储，多个`ziplist`之间使用双向指针串接起来  

可以看一下`quicklist`的源码  

```c
typedef struct quicklist { // src/quicklist.h
    quicklistNode *head;
    quicklistNode *tail;
    unsigned long count;        /* ziplist 的个数 */
    unsigned long len;          /* quicklist 的节点数 */
    unsigned int compress : 16; /* LZF 压缩算法深度 */
    //...
} quicklist;
typedef struct quicklistNode {
    struct quicklistNode *prev;
    struct quicklistNode *next;
    unsigned char *zl;           /* 对应的 ziplist */
    unsigned int sz;             /* ziplist 字节数 */
    unsigned int count : 16;     /* ziplist 个数 */
    unsigned int encoding : 2;   /* RAW==1 or LZF==2 */
    unsigned int container : 2;  /* NONE==1 or ZIPLIST==2 */
    unsigned int recompress : 1; /* 该节点先前是否被压缩 */
    unsigned int attempted_compress : 1; /* 节点太小无法压缩 */
    //...
} quicklistNode;
typedef struct quicklistLZF {
    unsigned int sz; 
    char compressed[];
} quicklistLZF;
```

![redis的quicklist](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/redis%E7%9A%84quicklist.png)  

`quicklist`内部默认单个`ziplist`长度为`8k`，超出这个字节数，就会新起一个`ziplist`，这个长度有配置参数`list-max-ziplist-size`决定  

那么`ziplist`又是什么呢？

简单的来说，`ziplist`是一种为了提高存储效率的特殊的双向链表。一个普通的双向链表，链表中的每一项都占用一块独立的内存，各项之间使用指针连接起来。这种方式会带来大量的内存碎片，而且地址指针也会占用额外的内存。而`ziplist`却是将表中的每一项存放在前后连续的地址空间内，一个`ziplist`整体占用了一大块内存。  

看一下`ziplist`的源码  

```c
struct ziplist<T> {
	int32 zlbytes; // 整个压缩列表占用字节数
	int32 zltail_offset; // 最后一个元素距离压缩列表起始位置的偏移量，用于快速定位到最后一个节点
	int16 zllength; // 元素个数
	T[] entries; // 元素内容列表，挨个挨个紧凑存储
	int8 zlend; // 标志压缩列表的结束，值恒为 0xFF
}
```

![redis中ziplist](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/redis%E4%B8%ADziplist.png)  

`ziplist`为了支持双向遍历，所以才有了`ztailf_offset`这个字段，用来快速定位最后一个元素，然后倒着遍历  

`ziplist`中的`entry`随着容纳的元素类型不同，也会有不一样的结构  

```c
struct entry {
	int<var> prevlen; // 前一个 entry 的字节长度
	int<var> encoding; // 元素类型编码
	optional byte[] content; // 元素内容
}
```

其中的`prevlen`表示的是前一个`entry`的长度，当`ziplist`倒着遍历的时候，需要通过这个字段来快速定位下一个元素的位置。它是一个边长的整数，当字符串长度小于254（`0XFE`）时，使用一个字节表示；如果长度达到或者超过了254（`0XFE`），那就使用5个字节来表示，第一个字节是`0XFE`（254），剩余的4个字节表示字符串长度  

`encoding`字段存储了元素内容的编码类型，`ziplist`通过这个字段来决定后面的`content`内容的形式  

![redis中ziplist的entry](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/redis%E4%B8%ADziplist%E7%9A%84entry.png)  

