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

## 哈希表    

哈希表，是将一个键值（`key`）和一个特殊的哈希表关联起来。这个关联起来的哈希表又包含两部分数：字段和值  

![redis哈希表](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/redis%E5%93%88%E5%B8%8C%E8%A1%A8.png)  

哈希对象的编码方式可以是`ziplist`或者是`hashtable`  

哈希对象保存的所有键值对的键和值的字符串长度都小于64字节并且保存的键值数对数量小于512个时，使用`ziplist`；否则使用`hashtable`  

如果使用`ziplist`的编码方式，此时的哈希表对象如下  

![redis中哈希表的ziplist](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/redis%E4%B8%AD%E5%93%88%E5%B8%8C%E8%A1%A8%E7%9A%84ziplist.png)  

使用`ziplist`的编码方式时，进行查找、删除、更新操作时会先定位到键的位置，然后再通过键的位置来确定值的位置  

如果使用`hashtable`的编码方式，总体上是使用数组+链表的结构。具体来说，是redis的字典（哈希表）底层使用哈希表实现，一个哈希表里面可以有多个哈希表节点，而每个哈希表节点就保存了字典（哈希表）中的一个键值对。下面分别介绍一下哈希表、哈希表节点和字典的实现

`redis`字典所使用的哈希表是由`dict.h/dictht`结构定义的

```c
typedef struct dictht {
    // 哈希表数组
    dictEntry **table;
    // 哈希表大小
    unsigned long size;
    // 哈希表大小掩码，用于计算索引值
    // 总是等于 size - 1
    unsigned long sizemask;
    // 该哈希表已有节点的数量
    unsigned long used;
} dictht;
```

`table`属性是一个数组，数组中的每个元素都是一个指向`dict.h/dictEntry`结构的指针，每个`dictEntry`结构保存着一个键值对  

`size`属性记录了哈希表的大小，也就是`table`数组的大小，而`used`属性则记录了哈希表目前已有节点（键值对）的数量  

`sizemark`属性的值总等于`size-1`，这个属性和哈希值一起决定一个键应该被放到`table`数组的那个所以上面  

下面展示的是一个大小为4的空哈希表（没有任何键值对）  

![redis空哈希表](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/redis%E7%A9%BA%E5%93%88%E5%B8%8C%E8%A1%A8.png)  

哈希表的节点是用`dictEntry`结构表示，每个`dictEntry`结构都保存着一个键值对  

```c
typedef struct dictEntry {
    // 键
    void *key;
    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;
    // 指向下个哈希表节点，形成链表
    struct dictEntry *next;
} dictEntry;
```

`key`属性保存着键值对中的键，`v`属性保存着键值对中的值，`next`属性是指向另一个哈希表节点的指针，这个指针可以将多个哈希值相同的键值对连接在一起，从而解决键冲突的问题  

举个例子，下图就展示了如何通过`next`指针，将两个索引值相同的键`k1`和`k2`连接在一起  

  ![redis解决hash冲突](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/redis%E8%A7%A3%E5%86%B3hash%E5%86%B2%E7%AA%81.png)

`redis`中的哈希表由`dict.h/dict`来表示  

```c
typedef struct dict {
    // 类型特定函数
    dictType *type;
    // 私有数据
    void *privdata;
    // 哈希表
    dictht ht[2];
    // rehash 索引
    // 当 rehash 不在进行时，值为 -1
    int rehashidx; /* rehashing not in progress if rehashidx == -1 */
} dict;
```

`type`属性和`privdata`属性是针对不同类型的键值对，为创建多态哈希表而设置的。`type`属性是一个指向`dictType`结构的指针，每个`dictType`结构保存了一簇用操作特定类型键值对的函数，`redis`会为用途不同的字典设置不同的类型特定函数；而`privdata`属性则保存了需要传给那些类型特定函数的可选参数  

`ht`属性是一个包含两个项的数组，数组中的每一项都是一个`dictht`哈希表，一般情况下，字典只是用`ht[0]`哈希表，`ht[1]`哈希表只会在对`ht[0]`哈希表进行`rehash`时使用  

此外还有一个`rehashidx`与`rehash`操作相关，它记录了`rehash`的进度，当`rehashidx`为-1时表示没有进行`rehash`  

因此`redis`中的哈希表结构如下  

![redis哈希表底层实现](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/redis%E5%93%88%E5%B8%8C%E8%A1%A8%E5%BA%95%E5%B1%82%E5%AE%9E%E7%8E%B0.png)

### `rehash`  

当元素数量等于数组的长度时，`redis`会进行扩容操作。`redis`开始执行`rehash`，这个过程分为3步  

1. 给哈希表`ht[1]`分配更大的空间  

2. 把哈希表`ht[0]`中的数据重新映射并拷贝到哈希表`ht[1]`中  

3. 释放哈希表`ht[0]`的空间  

这样有一个问题，当哈希表`ht[0]`的数据量很大，如果一次性复制就会造成线程阻塞，无法响应其他请求。这样是无法接受的，因此使用了渐进式`rehash`  

### 渐进式`rehash`  

上述进行`rehash`的时候，阻塞的地方可能发生在第二步。渐进式`rehash`则是在进行第二步的时候，仍然正常处理客户端的请求，每处理一个请求，顺带从哈希表`ht[0]`中第一个索引位置开始，把这个位置上的所有的`entry`复制到哈希表`ht[1]`，然后下一个请求就复制位置2；直到全部复制完成  

![渐进式rehash](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E6%B8%90%E8%BF%9B%E5%BC%8Frehash.png)  

具体过程如下  

1. 在字典中维持一个索引计数器变量`rehashidx`，并将其设置为0，表示开始进行`rehash`  

2. 在`rehash`期间，客户端每次对字典的进行`CRUD`操作时，会将`ht[0]`中`rehashidx`索引上的值`rehash`到`ht[1]`中，操作完成后`rehashidx+1`  

3. 字典操作不断执行，最终某个时间点，所有的键值对完成`rehash`，清空`ht[0]`表，将`ht[0]`和`ht[1]`的值进行互换，把新的`ht[1]`更名为`ht[0]`，这时将`rehashidx`设置为`-1`，表示`rehash`完成  

此外，`redis`还有一个定时任务在执行`rehash`，如果没有针对字典的请求，这个定时任务会周期性的搬迁一些数据到新的哈希表中  

在进行渐进式`rehash`的时候，如果要查找一个键，那么会优先去`ht[0]`中进行查找，如果不存在，再去`ht[1]`中进行查找。当执行新增操作时，新的键值对一律保存到新的`ht[1]`中，不再对`ht[0]`做任何操作，以保证`ht[0]`中的键值对数量只减不增，最后变为空表  

## 集合  

集合是一个无序并且拥有唯一的键值集合。集合对象的编码可以是`intset`也可以是`hashtable` 。集合对象保存的所有元素都是整数值并且保存的元素数量不超过512个，使用`intset`编码；否则使用`hashtable`编码  

当集合类型以`hashtable`存储时，哈希表的`key`为要插入的元素值，而哈希表的`value`为`null`  

![redis使用hashtable实现集合](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/redis%E4%BD%BF%E7%94%A8hashtable%E5%AE%9E%E7%8E%B0%E9%9B%86%E5%90%88.png)  

当集合的元素都是整数并且数量不多的时候，`redis`会使用`intset`来作为`set`的底层实现。源码如下  

```c
typeof struct intset {
    // 编码方式
    unit32_t encoding;
    // 集合包含的元素数量
    unit32_t lenght;
    // 保存元素的数组
    int8_t contents[];
} intset;
```

一个保存了5个整数的集合如下所示 

![redis intset](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/redis%20intset.png)  



   
