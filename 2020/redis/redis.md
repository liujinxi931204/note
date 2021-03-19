## redis支持的数据类型  
redis数据库支持五种数据类型  
1. 字符串(string)  
2. 哈希(hash)  
3. 列表(list)  
4. 集合(set)  
5. 有序集合(sorted set)  
### 字符串  
string是一组字节。在redis数据库中，字符串是二进制安全的。这意味它们具有已知的长度，并且不受任何特殊终止字符的影响。可以在一个字符串中存储最多512M字节的内容  
### 哈希  
哈希是键值对的集合。在redis中，哈希是字符串字段和字符串值之间的映射。因此，它们适合表示对象  
### 列表  
redis列表定义字符串列表，按插入顺序排序。可以将元素添加到redis列表的头部或者尾部  
### 集合  
集合(set)是redis数据库中的无序字符串集合。在redis中，添加、删除和查找的时间复杂度是O(1)  
### 有序集合  
redis有序集合类似于redis集合，也是一组非重复的字符串集合。但是，排序集合的每个成员都有一个分数与之相关联，该分数从最小到最高分数有序排序集。虽然成员是独特的，但是可以重复分数  
## redis命令  
redis命令用于在redis服务上执行操作。要在redis服务上执行命令需要一个redis客户端。在下载的redis安装包中  
### 语法  
启动客户端  
`redis-cli`  
该命令会连接本地的redis服务  
可以使用下面的命令检测redis服务是否启动  
```shell
redis-cli  
127.0.0.1:6379 >  
127.0.0.1:6379 > PING  
PONG  
```
如果需要在远程redis服务上执行命令，同样使用redis-cli命令  

### 语法  
`redis-cli -h host -p port -a password`  
可以使用下面的命令检测远程的redis服务是否启动   
```shell
redis-cli -h host -p port -a "password"  
127.0.0.1:6379 >  
127.0.0.1:6379 > PING  
PONG  
```
### 字符串  
字符串类型是redis最基础的数据结构。首先键都是字符串类型的，而且其他几种数据结构都是在字符串的基础之上构建的  
#### 常用命令    
1. 设置值  

`set key value [ ex seconds ] [px milliseconds ] [nx|xx]`  
时间复杂度O(1),将字符串值value关联到key，如果key已经持有其他值了，set旧覆写旧值，无视类型  
```shell
127.0.0.1:6379 > set hello world  
OK  
```
上面这句用于设置键为hello，值为world的键值对，返回结果OK表示设置成功  
set命令有几个选项  
ex seconds: 为键设置秒级过期时间  
px milliseconds: 为键设置毫秒级过期时间  
nx: 键必须不存在，才可以设置成功，用于添加  
xx: 键必须存在，才可以设置成功，用于更新，正好与nx相反  
此外，redis还提供setex和setnx两个命令    
用法如下  
`setnx key value`  
时间复杂度O(1),只有在键key不存在的情况下才能将键key的值设置为value；如果key已经存在，则不做任何操作。命令在设置成功时返返回1，设置失败时返回0    
`setex key seconds value`  
时间复杂度O(1)，将键key的值设置为value，并将键key的生存时间设置为seconds秒。如果键key已经存在，那么setex命令将覆盖已有的值。命令在设置成功时返回OK；在seconds参数不合法时，命令将返回一个错误  

2. 获取值  

`get key`  
时间复杂度为O(1),返回与键key相关联的字符串值。如果键key不存在，那么返回特殊值nil；否则返回键key的值  
如果键key的值并非字符串类型，那么返回一个错误，因为gei命令只能用于字符串值  
```shell
127.0.0.1:6379 > get hello  
"world"  
127.0.0.1:6379 > get db  
(nil)
```

3. 批量设置值 
   

`mset key value [key value ...]`  
时间复杂度O(n),	其中n为要被设置的键数  
`msetnx key value [key value ...]`  
时间复杂度为O(n),其中n为要被设置的键数  
mset与msetnx的区别在于  
**如果某个给定的键已经存在，mset将会使用新值覆盖旧值**
**如果某个给定的键已经存在，msetnx则会拒绝执行对所有键的设置操作**  
mset是一个原子性的操作，所有给定的键都会在同一时间内被设置，不会出现某些键被设置而另一些键没有被设置的情况；mset命令的返回值总是OK    
msetnx是一个原子性操作，所有给定的键要么同时被设置，要么全部不被设置，不会出现第三种状态；当给所有键设置成功时，返回值为1；如果因为某个键已经存在而导致未能设置成功，则返回0  
```shell
127.0.0.1:6379 > mset a 1 b 2 c 3 d 4  
OK
```

4. 批量获取  

`mget key [key ...]`  
时间复杂度为O(n),其中n为给定的键的数量。如果给定的字符串键里面有某个键不存在，那么这个键的值将以特殊值nil表示。  
结果是按照键传入顺序返回的     
```shell
127.0.0.1:6379 > mget a b c d  
1) "1"  
2) "2"  
3) "3"  
4) "4"  
```
**注意：批量操作所发送的命令不是无节制的，如果数量过多可能会造成redis阻塞或者网络阻塞**  

5. 获取并设置  

`getset key value`  
时间复杂度为O(1),将键key的值设置为value，并将建key的旧值返回  
如果键key没有旧值，则返回nil;如果键key不是字符串类型时，则返回一个错误  

6. 计数  

`incr key`  
时间复杂度为O(1),为键key存储的数字值加1  
如果键key不存在，那么它的值会先被初始化为0，然后再执行incr命令,返回1；如果键存储的值是整数，返回自增后的结果；如果键存储的值不能被解释为数字，那么incr命令将会返回一个错误  
INCR命令是一个针对字符串的操作。因为redis并没有专用的整数类型，所以键key存储的值在执行incr命令时会被解释为十进制64位有符号整数  

`incrby key increment`  
时间复杂度为O(1)，为键key存储的数字值加上增量increment  
如果键key不存在，那么他的值会先被初始化位0，然后再执行incr命令，返回increment的值；如果键存储的值时整数，返回增加increment后的值；如果键key存储的值不能被解释位数字，那么该命令返回一个错误  

`decr key`  
时间复杂度O(1),为键key存储的数字值减1  
如果键key不存在，那么它的值会先被初始化为0，然后执行decr命令，返回-1；如果键key存储的值是整数，返回自减后的结果；如果存储的值不能被解释为数字，那么decr命令将会返回一个错误  

`decrby key decrement`  
时间复杂度为O(1),为键key存储的数字值减increment  
如果键key不存在，那么他的值会先被初始化为0，然后执行decrby命令，返回减法之后的结果；如果键key存储的是整数，返回执行减法操作之后的值；如果键key存储的值不能被解释为数值，那么decrby命令将会返回一个错误  
### 内部编码  
字符串类型的内部编码有3种：
+ int：8个字节的长整型  
+ embstr： 小于等于39个字节的字符串  
+ raw: 大于39个字节的字符串  
redis会根据当前值的类型和长度决定使用哪种内部编码实现  
### 哈希  
redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象，每个hash键中可以存储多达40亿个字段值  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/09/1599631827179-1599631827213.png)  
hash类型中的映射关系叫做field-value，注意这里的value是指field对应的值，不是键对应的值  
#### 常用命令  
1. 设置值  

`hset hash field value`  
时间复杂度为O(1)，将哈希表hash中域field的值设置为value。如果给定的哈希表不存在，那么会一个新的哈希表将被创建并执行hset操作；如果域field已经存在于哈希表中，那么它的旧值将被新值value覆盖  
当hset命令在哈希表中创建新的field域并成功为它设置值时，命令返回1；如果域field已经存在域哈希表中并且hset命令成功使用新值覆盖了它的旧值，那么命令返回0  
`hsetnx hash field value`  
时间复杂度为O(1)，当且仅当field尚未存在于哈希表的情况下，将它的值设置为value；如果给定域已经存在于哈希表中，那么将放弃执行设置操作；如果哈希表不存在，那么一个新的哈希表将被创建并执行hsetnx操作  
hsetnx命令在设置成功时返回1，在给定域已经存在而放弃执行设置操作返回0  
`hmset key field vlaue [ field value ...]`  
时间复杂度O(n),n为field-value对的数量，同时将多个field-value对设置到哈希表hash中，此命令会覆盖哈希表中已存在的域。如果key不存在，一个空哈希表被创建并执行mhset操作  
如果命令执行成功，返回"OK";当key不是哈希表类型时，返回一个错误  

2. 获取值  

`hget hash field`  
时间复杂度为O(1)，返回哈希表中给定域的值。hget命令在默认情况下返回给定域的值，如果给定域不存在于哈希表中，又或者给定的哈希表不存在，那么命令返回nil  
`hmget key field [ field ...]`  
时间复杂度为O(1)，返回哈希表中key中，一个或多个给定域的值。如果给定的域不存在域哈希表中，那么返回一个nil。因为不存在的哈希表key被当作一个空表来处理，所以对一个不存在的key进行hmget操作将返回一个只带有nil值的表  
`hgetall key`  
时间复杂度为O(n)，返回hash表中所有的域和值，在返回值里，紧跟在每个域名filed之后时域的值，所以返回值的长度是哈希表大小的两倍  
以列表的形式返回哈希表的域和域的值，若key不存在，返回空列表  
在使用hgetall时，如果哈希表中元素比较多，有可能会阻塞redis。如果只是获取部分field，可以使用hmget；如果一定要获取全部field-value，可以使用hscan，该命令会渐进式遍历哈希类型  

3. 删除field  
   

`hdel key field [ field ...]`  
时间复杂度为O(n)，n为要删除的域的数量，删除哈希表中的一个或多个指定域，不存在的域将被忽略。返回成功被移除的域的数量，不包括被忽略的域  

4. 返回域的数量  

`hlen key`  
时间复杂度O(1)，返回哈希表key中域的数量，当key不存在时返回0  

5. 域是否存在  

`hexists hash field`  
时间复杂度为O(1)，检查给定域filed是否存在于哈希表hash中，如果给定的域存在则返回1，否则返回0  

6. 获取所有field  
   

`hkeys key`  
时间复杂度为O(n),n为哈希表的大小，返回哈希表key中所有的域的表，当key不存在时，返回空表  

7. 获取所有的value  

`hvals key`  
时间复杂度为O(n),n为哈希表的大小，返回哈希表key中所有的域的值的表，当key不存在时，返回空表  

8. 自增、自减  

`hincrby key field increment`  
时间复杂度为O(1)，为哈希表key中域field的值增加increment。增量也可以是负数，相当于对给定域做减法。如果key不存在，一个新的哈希表将被创建并执行hincrby命令；如果域field不存在，那么在执行命令前域的值被初始化为0，然后执行hinrby操作；对于一个存储字符串的域field执行hincrby命令将会造成一个错误  

9. 计算value的字符串长度  

`hstrlen key field`  
时间复杂度为O(1),返回哈希表中key中给定域field相关联的值的字符串的长度。如果指定的键或者域不存在，那么返回0  
#### 内部编码  
哈希类型的内部有两种编码  
+ ziplist(压缩列表)：当有value大于64字节或者field的个数超过512时，内部编码会由ziplist变为hashtable。redis会使用ziplist作为哈希的内部实现，ziplist使用更加紧凑的结构实现多个元素的连续存储，所以在节省内存方面比hashtable更加优秀  
+ hashtable(哈希表):当哈希类型无法满足ziplist的条件时，redis会使用hashtable作为哈希的内部实现，因为此时ziplist的读写效率会下降，而hashtable的读写时间复杂度为O(1)  
### 列表  
列表(list）类型是用来存储多个有序字符串，列表中的每个元素成为element，一个列表最多可以存储2^32^-1个元素。在redis中，可以对列表两端插入(push)和弹出(pop)，还可以获取指定范围的元素列表、获取指定索引下标的元素等  
列表类型有两个特征：1. 列表中的元素是有顺序的，这就意味着可以通过索引下标获取某个元素或者某个范围内的元素列表；2. 列表中的元素可以是重复的  
#### 常用命令  
1. 插入  

`lpush key value [ vlaue ...]`  
时间复杂度为O(1),将一个或多个值插入到列表key的表头（最左边），如果有多个值，那么每个value值按照从左到右的顺序插入到列表头；例如执行`lpush mylish a b c`，列表的值将会是c、b、a。该命令等同于原子性地操作`lpush mylist a` `lpush mylist b` `lpush mylist c`  
如果key不存在，一个空列表将被创建并执行lpush操作；如果key存在但不是列表类型时，则会返回一个错误  

`lpushx key value`  
时间复杂度为O(1),将值value插入到列表可以的表头，当且仅当key存在并且是一个列表。当key不存在，lpushx不做任何操作  

`rpush key value [ value ...]`  
时间复杂度为O(1),将一个或多个值value插入到列表key的表为(最右边),如果有多个value值，那么各个value值将按照从左到右的顺序依次插入到表尾；如果key不存在，一个空列表会被创建并执行rpush操作；当key存在但不是列表类型时，返回一个错误  

`rpushx key value`
时间复杂度为O(1),将值value插入到列表key的表为尾，当且仅当key存在并且是一个列表时；当key不存在，rpush不会做任何操作  

`linsert key before|after pivot value`  
时间复杂度为O(n),n为寻找pivot过程中经过的元素的数量  
将值value插入到列表key当中，位于值pivot之前或之后；如果pivot不存在于列表key时，不执行任何操作；当key存在但不是列表类型时，key被视为空列表，不执行任何操作；如果key不是列表类型时，返回一个错误  
如果命令执行成功，返回插入操作完成后列表的长度；如果没有找到pivot，返回-1；如果key不存在或者为空时，返回0  

2. 查  

`lrange key start stop`  
时间复杂度为O(s+n),s为偏移量start、n为指定区间内元素的数量；返回列表key中指定区间内的元素区间以偏移量start和stop指定  
下标参数start和stop都从0开始，也可以使用负数，-1表示列表的最后一个元素，-1表示列表的倒数第二个元素...  
在redis执行`lrange mylist 0 10`会返回一个包含11个元素的列表，这与一般的编程语言有所区别  
超出范围的下标不会引起错误：如果start下标比列表的最大下标end(列表长度-1)还要大，返回一个空列表；如果stop下标比最大下标end(列表长度-1)还要大，redis将stop的值设置为end  

`lindex key index`  
时间复杂度为O(n),n为达到下标index的过程中经过元素数量。因此，对表头和表尾元素执行lindex命令，时间复杂度为O(1)  
返回列表key中，下标为index的元素  
如果key不是列表元素，返回一个错误；如果index不在列表区间范围内，返回nil  

`llen key`  
时间复杂度为O(1),返回列表key的长度；如果key不存在，则key被解释为一个空列表，返回0；如果key不是列表类型，返回一个错误  

3. 删除  

`lpop key`  
时间复杂度为O(1),移除并返回列表key的头元素，当列表key不存在时，返回nil  

`rpop key`  
时间复杂度为O(1),移除并返回列表key的尾元素，当列表key不存在时，返回nil  

`ltrim key start stop`  
时间复杂度为O(n),n为被移除的元素的数量  
对一个列表进行修剪，就是说，让列表只保留指定区间范围内的元素，不在指定区间之内的元素都被删除  
例如执行命令`ltrim mylist 0 2`,表只保留列表list的钱三个元素，其余元素全部删除  
当key不是列表类型时，返回一个错误  
超出范围不会引起错误，如果start下标比列表的最大下标end(列表长度-1)或者start>stop,,ltrim返回一个空列表，因为此时ltrim已经将整个列表情况  
如果stop下标比end(列表长度-1)还要大，redis将stop设置为end  

`lrem key count value`  
时间复杂度为O(n)，n为列表的长度，根据参数count的值，移除列表中与参数value相等的元素  
count的值可以是以下几种  
count>0:从表头开始向表尾搜索，移除与value相等的元素，数量为count  
count<0:从表尾开始向表头搜索，移除与value相等的元素，数量为count的绝对值  
count=0:移除表中所有与value相等的值  
当key不存之时，视为空表，lrem返回0  

4. 设置  

`lset key index value`  
时间复杂度：对表头元素或表尾元素进程lset操作，时间复杂度为O(1);其他情况下O(n),n为列表的长度。将列表key中下标为index的元素设置为value；当index查出范围或者对一个空列表进行lset操作时，返回一个错误  

5. 阻塞操作  

`blpop key [ key ...] timeout`  
时间复杂度为O(1),blpop是列表的阻塞式弹出原语，是lpop的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被blpop阻塞，直到等待超时或者发现可弹出元素为止  
当给定多个key参数时，按照参数的先后顺序依次检查各个列表，弹出第一个非空列表的头元素  
如果列表为空，返回一个nil；否则返回一个含有两个元素的列表，第一个元素是被弹出元素所属的key，第二个元素是被弹出的元素的值  
超时参数如果为0，表示阻塞的时间可以无限延长  

`brpop key [key...] timeout`  
时间复杂度为O(1),brpop是列表阻塞式弹出原语，是brpop的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被blpop阻塞，直到等待超时或者发现可弹出元素为止  
当给定多个key参数时，按照参数的先后顺序依次检查各个列表，弹出第一个非空列表的尾元素  
如果列表为空，返回一个nil；否则返回一个含有两个元素的列表，第一个元素是被弹出元素所属的key，第二个元素是被弹出的元素的值  
超时参数如果为0，表示阻塞

6. 复合操作  

`rpoplpush source destination`  
时间复杂度为O(1),命令rpoplpush在一个原子时间内，执行以下两个动作：  
将列表source中的最后一个元素(表尾元素)弹出，返回给客户端；
将source弹出的元素插入到列表destination，作为列表destination的表头  
如果source不存在，值nil被返回，并且不执行其他任何动作  
如果source、destinatoin相同，则列表中的表尾元素被移动到表头，并返回该元素，可以把这种特殊情况视为列表的旋转操作  

`brpoplpush source destination timeout`  
时间复杂度为O(1),brpoplpush是rpoplpush的阻塞版本，当列表source不为空时，和rpoplpush表现一致  
当列表source为空时，brpoplpush命令将阻塞连接，直到等待超时，或者有一个客户端对source执行了lpush或rpush操作为止  
超时参数如果为0，表示阻塞时间可以无限期延长  
#### 内部编码  

列表类型内部有两种编码  
+ ziplist:当列表的元素个数小于list-max-ziplist-entries配置(默认512个)同时列表中每个元素的值都小于list-max-ziplist-value配置时(默认64字节),redis会选用ziplist来作为列表内部实现来减少内存的使用  
+ linkedlist:当列表类型无法满足ziplist的条件时，redis会使用linkedlist作为列表内部实现  
  
### 集合  
集合(set)类型也是用来保存多个的字符串元素，但是和列表类型不一样的是，集合中不允许有重复的元素，并且集合中的元素是无序的，不能通过索引下标获取元素  
一个集合最多可以存储2^32^-1个元素。redis除了支持集合内的增删改查，同时还支持多个集合去交集、并集、差集  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/14/1600064807779-1600064807866.png)  
#### 常用命令  

1. 添加元素  

`sadd key member [ member ...]`  
时间复杂度为O(n)，n是被添加的元素的数量  
将一个或多个member元素加入到集合key当中，已经存在于集合key中的元素将被忽略；假如key不存在，则创建一个只包含member元素作成员的集合，当key不是集合类型时，返回一个错误  
返回被添加到集合中的新元素的数量，不包括被忽略的元素  

2. 删除元素  

`srem key memeber [member...]`  
时间复杂度为O(n),n为给定member元素的数量  
移除集合key中的一个或多个member元素，不存在的member元素会被忽略；当key不是集合类型，返回一个错误  
返回被成功移除的元素的数量，不包括被忽略的元素  

3. 计算元素数量  

`scard key`  
时间复杂度为O(1),返回集合key的基数（集合中元素的数量），当key不存在时，返回0  
它不会遍历整个集合，而是直接去用redis的内部变量  

4. 判断元素是否在集合中  

`sismember key member`  
时间复杂度为O(1),判断member元素是否是集合key的成员，如果member元素是集合的成员，返回1；如果member元素不是集合的成员，或者集合key不存在，返回0  

5. 随机从集合中返回指定个数元素  

`srandmember key [count]`  
时间复杂度为：如果只提供了key参数时，时间复杂度为O(1)如果提供了count参数，那么为O(n),n为返回数组的个数  
如果命令执行时，只提供了key参数，那么返回集合中的一个随机元素  
如果count为正数，且小于集合基数，那么命令返回一个包含count个元素的数组，数组中的元素**各不相同**；如果count大于等于集合基数，那么返回整个集合  
如果count为负数，那么命令返回一个数组，数组中的元素**可能会重复出现多次**，而数组的长度为count的绝对值  
只提供key参数时，返回一个元素；如果集合为空，返回nil；如果提供了count参数，返回一个数组；如果集合为空，返回空数组  

6. 从集合中随机弹出元素  

`spop key`  
时间复杂度为O(1),移除并返回一个随机的元素  
返回被移除的随机元素，当key不存在或key是空集合时，返回nil  
**注意：spop key是随机移除一个元素，并返回；srandmember key是随机返回一个元素，并不对原来的集合做任何操作**  

7. 获取所有元素  

`smembers key`  
时间复杂度为O(n),n为集合的基数，返回集合key中的所有成员，不存在的key视为空集合  

8. 交集  

`sinter key [key...]`  
时间复杂度为O(n*m),n为给定集合中基数最小的集合,m为集合的个数  
返回一个集合的全部成员，该集合是所有给定集合的交集，当给定集合当中有一个空集时，返回空集，不存在的key视为空集  

`sinterstore destnation key [key...]`  
时间复杂度为O(n*m),n为给定集合中基数最小的集合,m为集合的个数  
将交集的结果保存在destnation中，而不是简单的返回；如果destnation集合已经存在，则将其覆盖；destnation可以是key本身    

9. 并集  

`sunion key [key...]`  
时间复杂度为O(n),n为所有给定集合的元素数量之和  
返回一个集合的全部成员，该集合是所有给定集合的并集  
不存在的key被视为空集  

`sunionstore destnation key [key...]`  
时间复杂度为O(n),n为所有给定集合的元素数量之和  
将并集的结果返回到destnation集合，而不是简单的返回；如果destnation已经存在，则将其覆盖；destnation可以是key本身  

10.  差集  

`sdiff key [key...]`  
时间复杂度为O(n),n是所有集合的成员数量之和  
返回一个集合的全部成员，该集合是所有给定集合之间的差集  
不存在的key被视为空集  

`sdiffstore destnation key [key...]`  
时间复杂度为O(n),n是所有集合的成员数量之和  
将结果保存到destnation中，而不是简单的返回；如果destnation已经存在，将会被覆盖；destantion可以是key本身  

#### 内部编码  
集合类型的内部编码有两种：  
+ intset(整数集合):当集合中元素都是整数且元素个数小于set-max-inrset-entries配置(默认512个)时，redis会选用intset来作为集合内部的实现，从而减少内存使用  
+ hashtable(哈希表):当集合类型无法满足intset的条件时，redis会使用hashtable作为集合的内部实现  
  
### 有序集合  
有序集合保留了集合不能有重复成员的特性，但是不同的是，有序集合中的元素可以排序。但是它和列表使用索引下标作为排序依据不同的是，它给每一个元素设置一个分数(sorce)作为排序的依据  
有序集合中的元素不能重复，但是score可以重复  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/14/1600069670857-1600069670858.png)  

#### 常用命令  
1. 添加成员  

`zdd key [NX|XX] [CH] [INCR] score member [[sorce member] [sorce member]..]`  
时间复杂度为O(m*log(n)),n是有序集合的基数，m为成添加的新成员的数量  
将一个或多个member元素以及score值成功加入到有序集合key当中  
当某个member已经是有序集的成员，那么更新这个member的score值，并通过重新插入这个member元素，来保证该member在正确的位置上  
如果key不存在，则创建一个空的有序集并执行zadd操作；当key存在但不是一个有序集时，返回一个错误  
zadd有四个选项  
+ nx:member必须不存在，才可以设置成功，用于添加  
+ xx:member必须存在，才可以设置成功，用于更新  
+ ch:返回此次操作后，有序集合元素和分数发生变化的个数  
+ incr:对sorce做增加，相当于后面介绍的zincrby  
  
2. 计算成员个数  

`zcard key`  
时间复杂度为O(1),返回有序集合的key的基数，当key存在并且时有序集合类型时，返回有序集合的基数；如果key不存在，返回0  

3. 计算某个成员的分数  

`zscore key member`  
时间复杂度为O(1),返回有序集key中成员member的score值  
如果member元素不是有序集合key的成员，或key不存在时，返回nil  

4. 计算成员的排名  

`zrank key member`  
`zreverank key member`  
时间复杂度为O(log(n)),返回有序集key中成员member的排名。zrank按照score的值从小到大排列；zrevrank按照score的值从大到小排列；排名从0开始  
如果有序集key的成员member存在，返回排名；如果member不是有序集key的成员，返回nil  

5. 删除成员  

`zrem key member [member...]`  
时间复杂度为O(m*log(n))，n为有序集合的基数，m为被成功移除的成员的数量  
移除有序集key中的一个或多个成员，不存在的成员将会被忽略；当key存在都不是有序集合类型时，返回一个错误  

6. 增加成员分数  

`zincrby key increment member`  
时间复杂度为O(log(n))，为有序集合key的成员member的score值上增加量increment，可以通过增加一个负值实现减去的功能  
当key不存在或member不是key的成员时，zincrby key increment member会转化为zadd key increment member；当key不是一个有序集合类型时，返回一个错误，score的值可以是整数或者双精度浮点数  

7. 返回执行排名范围内的成员  

`zrange key start end [withscores]`  
`zreverange key start end [withscores]`  
时间复杂度为O(log(n)+m),n为有序集合的基数，m为结果集的基数  
返回有序集key中，指定区间的成员；zrange按照socre值递增来排序；zreverange按照score值递减来排序，具有相同score值的成员按字典序来排列  
下标参数start和stop都以0为底，超出范围的下标并不会引起错误。当start的值比有序集合的最大下标还有大或者start>stop时，会返回一个空列表；如果stop参数的值超过了有序集合最大下标，那么redis将stop当作最大下标来处理  
可以通过使用withscores选项来让成员和它的score值一并返回  

8. 返回指定分数范围的成员  

`zrangebyscore key min max [withscores] [limit offset count]`  
`zreverangebyscore key min max [withscores] [limit offset count]`  
时间复杂度为O(log(n)+m),n为有序集合的基数，m为被结果集的基数  
返回有序集合key中，所有score值介于min和max之间(包括min和max)的成员。zrangebyscore按照score值递增来排序；zreverangebyscore按照score值降序来排序；具有相同score值的成员按字典序来排列  
可选的limit参数指定返回结果的数量及区间，注意当offset很大时，定位offset的操作可能需要套遍历整个有序集，此过最坏复杂度为O(n)时间  
可选的withscores参数决定结果是单单返回有序集的成员还是将有序集的成员及其score值一起返回  

9. 删除指定排名内的元素  

`zremrangebyrank key start stop`  
时间复杂度为O(log(n)+m),n为有序集的基数，m为被移除成员的数量  
移除有序集key中，指定排名rank区间内的所有成员；区间分别以下标参数start、stop指出，包含start和stop在内，下标都是以0开始的  

10. 删除指定分数范围内的成员  

`zremrangebyscore key min max`  
时间复杂度为O(log(n)+m),n为有序集的基数，m为被移除成员的数量  
移除有序集key中的所有score值介于min和max之间（包括min和max）的成员  

11. 交集  

`zinterstore destnation numkeys key [key...] [weights weight [weight...]] [aggregate sum|min|max]`  
`时间复杂度为O(n*k)+O(m*log(n))`,n为给定key中基数的最小的有序集，k为给定有序集数量，m为结果集的基数  
计算给定的一个或多个有序集的交集，其中给定key的数量必须以numkeys参数指定，并将该交集存储到destnation  
默认情况下，结果集中某个成员的score值是所有给定集下该成员score值之和  
返回保存到destnation的结果集的基数  
+ numkeys：需要做交集计算键的个数  
+ weighths weight [weight...]:每个键的权重，在做交集计算的时候，每个键中每个member会将自己score乘以这个权重，每个键的默认权重是1  
+ aggregrate sum|min|max:计算成员交集后，score可以按照sum、min、max做汇总，默认是sum  
  
12. 并集  

`zunionstore destnation numkeys key[key...] [weights weight[weight...]] aggregrate[sum|min|max]`  
时间复杂度为O(n)+O(mlog(n))  
计算给定的一个或多个有序集的并集，其中给定的key的数量必须以numkeys参数指定，并将该结果存储到destnation中  
参数的意义同上  
#### 内部编码  
+ ziplist：当有序集合的元素个数小于zset-max-ziplist-entries(默认128个),同时每个元素的值都小于zset-max-ziplist-value配置(默认64字节),redis会使用ziplist作为有序集合的内部实现，ziplist可以有效减少内存的使用  
+ skiplist：当ziplist条件不满足时，有序集合会使用skiplist作为内部实现，因为此时ziplist的读写效率会下降  
## 键管理  
1. 判断键是否存在  

`exists key`  
时间复杂度为O(1),检查给定的key是否存在，如果key存在返回1，否则返回-1  

2. 键的类型  

`type key`  
时间复杂度为O(1),返回key所存储的值的类型  

3. 重命名  

`renanme key newkey`  
时间复杂度为O(1)，将key改名为newkey，当key和newkey相同，或者key不存在时，返回一个错误；当newkey已经存在时，rename命令将覆盖旧值  

`renamenx key newkey`  
时间复杂度为O(1),当且仅当newkey不存在时，将key改名为newkey，返回1；当key不存在时，返回一个错误；当newkey存在时，返回0  

**由于重命名键期间会执行del操作命令删除旧键，如果键对应的值比较大，会存在阻塞redis的可能性 ** 

4. 移动  

`move key db`  
时间复杂度为O(1),将当前数据库的key移动到给定的数据库db当中  
如果当前数据库(源数据库)和给定数据库(目标数据库)有相同名字的给定key，或者key不存在于当前数据库中，那么move没有任何效果  

5. 删除  
`del key [key...]`  
时间复杂度为O(n),n为被删除的key的数量，其中删除单个字符串类型的key，时间复杂度为O(1);删除单个列表、集合、有序集合或哈希表类型的key，时间复杂度为O(m)，m为以上数据结构内的元素数量  
  
6. 随机返回一个键  

`randomkey`  
时间复杂度为O(1),从当前数据库中随机返回一个(不删除)key，当前数据库不为空时，返回一个key；为空时，返回nil  
## 键过期  
除了expire、ttl之外，redis还提供了expireat、pexpire、pexpireat、pttl、persist等一系列命令  

`expire key seconds` 键在seconds秒后过期  
`expireat key timestamp` 键秒级的时间戳timestamp后过期  
ttl和pttl都可以查询键的剩余过期时间，但是pttl精度更高可以达到毫秒级，有3种返回值  
+ 大于等于0的整数：键的剩余过期时间(ttl是秒，pttl是毫秒)  
+ -1：键没有设置过期时间  
+ -2：键不存在  
  

`pexpire key milliseconds` 键在milliseconds毫秒后过期  
`expireat key millisecondstamp` 键毫秒级的时间戳timestamp后过期  
无论使用过期时间还是过期时间戳，秒级还是毫秒级，在redis内部最终都是使用pexpireat  
+ 如果expire key的键不存在，返回结果为0  
+ 如果过期时间为负值，建会被立即删除，犹如使用del一样  
+ persist命令可以将键的过期时间清楚  
+ 对于字符类型键，执行set命令会去掉过期时间  
+ redis不支持二级数据结构(如哈希、列表)内部元素的过期功能  
+ setex命令作为set+expire的组合，不但是原子执行，同时减少了一次网络通讯的时间  
## 迁移键  
redis发展历程中提供了move、dump+restore、migrate三组迁移键的方法  
+ move  
`move key db`  
move命令用于在redis内部进行数据迁移。`move key db`就是把指定的键从源数据库移动到目标数据库中  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/15/1600141652846-1600141652916.png)  
+ dump+restore  
```shell
dump key
restore key ttl value
```
dump+restore可以实现在不同redis实例之间进行数据迁移的功能，整个迁移的过程分为两步：  
1) 在源redis上，dump命令会将键值序列化，格式采用RDB格式  
2) 在目标redis上，restore命令将上面序列化的值进行复原，其中ttl代表整个过期时间，如果ttl=0代表没有过期时间  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/15/1600141868951-1600141868952.png)  
有关dump+restore有两点需要注意：第一，整个迁移过程并非原子性的，而是通过客户端分布完成的；第二，迁移过程是开启了两个客户端连接，所以dump的结果不是源redis和目标redis之间进行传输  
+ migrate  
`migrate host port key|"" destnation-db timeout [copy] [replace] [keys key ...]`  
host：目标redis的ip地址    
port：目标redis的端口  
key|""：如果需要迁移一个键，此处为要迁移的键；如果要迁移多个键，此处为空字符串  
destnation-db：目标redis的数据库索引  
timeout：迁移的超时时间  
[copy]：如果添加此选项，迁移后并不删除源键  
[replace]：如果添加此选项，migrate不管目标redis是否存在该建都会进行正常迁移进行数据覆盖  
[keys key ...]：迁移多个键，例如要迁移"key1、key2 key3"，此处填写"keys key1 key2 key3"  
migrate命令也是用于在redis实例之间进行数据迁移的，实际上migrate命令就是将dump、restore、del三个命令进行组合，从而简化了操作流程，migrate命令具有原子性  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/15/1600150609943-1600150609945.png)  
整个过程和dump+restore基本类似，但是有3点不同  
1）整个过程是原子执行的，不需要在多个redis实例上开启客户端的，只需要在源redis上执行migrate命令即可  
2）migrate命令的数据传输直接在源redis和目标redis上完成的  
3）目标redis完成restore后会发送ok给源redis，源redis接收后会根据migrate对应的选项来决定是否在源redis上删除对应的键  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/15/1600151427009-1600151427010.png)  
## 遍历键  
redis提供了两个命令来遍历所有的键  
+ keys 
`keys pattern`  
实际上keys命令是支持pattern匹配的  
当需要遍历所有键时(例如检测过期或限制时间、寻找大对象),keys是一个很有帮助的命令，如果redis包含了大量的键，执行keys命令很有可能会造成redis阻塞  

+ scan  
scan采用渐进式遍历的方式来解决keys命令可能带来的阻塞问题，每次scan命令的时间复杂度为O(1),但实际要真正实现keys的功能，需要多次执行scan  
`scan cursor [match pattern] [count number]`  
cursor是必须参数，实际上cursor是一个游标，第一次遍历从0开始，每次scan遍历完都会返回前游标的值，直到游标值为0，表示遍历结束  
[match cursor]是可选参数，它的作用是做模式匹配  
[count number]是可选参数，它的作用是表明每次要遍历的键的个数，默认值是10，此参数可以适当增大  
## 数据库管理  
1. 切换数据库  
`select dbIndex`  
redis默认配置中是有16个数据库，当使用redis-cli -h{ip} -p{port}连接redis时，默认使用的就是0号数据库，当选用其他数据库时，会有[index]的前缀表示，这里index就是数据库的索引下标  
不同数据库之间没有任何关联，甚至可以存在相同的键  
2. flushdb/flushall  
flushdb/flushall命令用于清除数据库，两者的区别在于flushdb只清除当前数据库，flushall会清除所有数据库  
如果当前数据库键值数量比较多，flushdb/flushall存在阻塞redis的可能性  