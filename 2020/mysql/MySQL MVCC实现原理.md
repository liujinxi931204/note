## 什么是MVCC  
MVCC，全称Multi-Version Concurrency Control,即多版本并发控制。MVCC是一种并发控制的方法，一般在数据库管理系统中，实现对数据库的并发访问，在编程语言中实现事务内存  
MVCC在MySQL InnoDB存储引擎中实现主要是为了提高数据库并发访问性能，用更好的方式去处理读-写冲突，即做到即使有读写冲突时，也能做到不加锁，非阻塞并发读  
## 什么是快照读和当前读  
### 当前读  
像`select lock in share mode、select for update、insert、update、delete`这些操作就是当前读，即读取的是记录的最新版本，读取时还要保证其他并发事务不会修改当前记录，会对读取的记录加锁
### 快照读  
不加锁的读就是快照读，即普通不加锁的select语句就是快照读，也就是不加锁的非阻塞读。快照读的前提是隔离级别不能是串行化，在串行化隔离级别下，快照读会退化为当前读。之所以出现快照读，是基于提高并发性能的考虑，快照读的实现是基于MVCC。在很多情况下，MVCC避免了加锁的操作，降低了开销；既然是基于多版本并发控制，即快照读可能读到的并不是数据的最新版本而有可能是数据某一历史版本  
**总之，MVCC就是为了实现读-写不冲突，而这个读指的就是快照读而不是当前读。当前读实际上是一种加锁的操作，是悲观锁的实现**  
## MVCC能解决什么问题  
**数据库并发场景有三种**：  
1. 读-读：不存在任何问题，也不需要并发控制  
2. 读-写：有线程安全问题，可能会造成事务隔离性问题，可能遇到脏读、幻读、不可重复读  
3. 写-写：有线程安全问题，可能存在更新丢失问题。比如第一类更新丢失、第二类更新丢失  
多版本并发控制(MVCC)是一种解决读-写冲突的无锁并发控制。在并发读写时，可以做到读操作不用阻塞写操作、写操作不用阻塞读操作；同时还可以解决脏读、幻读、不可重复读等事务个理性问题，但不能解决更新丢失问题  
## MVCC实现原理  
MVCC的目的就是多版本并发控制，在数据库的实现中，就是为了解决读写冲突，它的实现原理主要依赖记录中的**3个隐式字段、undo log日志、Read View**来实现的  
### 隐式字段  
MySQL除了每行记录除了用户自已定义的字段外，还会有**DB_TRX_ID、DB_ROLL_PTR、DB_ROW_ID**等字段  
1. DB_TRX_ID  
6 byte，最近修改(更新、插入)事务ID：记录创建这条记录\最后一次修改该记录的事务ID  
2. DB_ROLL_PTR  
7 byte，回滚指针，指向这条记录的上一个版本(存储与rollback segment里)  
3. DB_ROW_ID  
6 byte，隐含的自增ID(隐藏主键)，如果数据库没有主键，InnDB会自动以DB_ROW_ID产生一个聚簇索引  
实际上还有一个删除flag隐藏字段，即记录被删除不代表真的删除，而是删除flag变了  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/04/1599202238249-1599202238349.png)  
如上图，DB_ROW_ID是数据库默认为该行记录生成的唯一隐式主键，DB_TRX_ID是当前操作该记录的事务ID，DB_ROLL_PTR是一个回滚指针，用于配合undo log，指向上一个版本  
### undo log日志  
undo log主要有两种：
1. insert undo log
代表事务在insert新纪录时产生的undo log只在事务回滚时需要，并且在事务提交以后可以被立即丢弃  
2. update undo log：
事务在进行update或者delete时产生的undo log，不仅在事务回滚时需要，在快照读时也需要，所以不能随便删除，只有在快照读或者事务回滚不涉及该日志时，对应的日志才会被purge线程统一清楚  
对MVCC有帮助的实际时update undo log，undo log实际上就是存在rollback segment中旧链表  
一、比如一个事务在persion表中插入了一条新纪录  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/04/1599204450085-1599204450089.png)  
二、现在来了一个事务1对该记录的name做出了修改，改为Tom
1. 在事务1修改该行记录时，数据库会先对该行加锁  
2. 然后把该行记录拷贝到undo log中，作为旧记录，即在undo log中有当前行的拷贝副本  
3. 拷贝完毕以后，修改该行name为Tom，并且修改隐藏字段的事务为当前事务1的ID，这里假设从1开始之后递增；回滚指针指向拷贝到undo log中的记录副本，表示我的上一个版本就是它  
4. 事务提交，释放锁  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/04/1599204859793-1599204859795.png)  
三、这时又有一个事务2对该记录的age做出了修改，改为30  
1. 在事务2修改该行记录时，数据库会先对该行记录加锁  
2. 然后把该行记录拷贝到undo log中，作为旧记录，发现该行记录已经有了undo log，那么最新的旧数据作为链表的表头，插在该行记录的undo log最前面  
3. 修改该行记录的age为30，并且修改隐藏字段的事务ID为当前事务2的ID，就是2；回滚指针指向刚刚拷贝到undo log的副本记录  
4. 事务提交，释放锁  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/04/1599205250465-1599205250466.png)  
从上面可以看出，不同事务或者相同事务对同一记录的修改，会导致该记录的undo log成为一条记录版本链表，undo log的表头就是最新的旧记录，表尾就是最早的旧记录。(当然，undo log的节点会被purge线程清理掉)  
### Read View  
#### 什么是Read View  
Read View就是事务进行快照读操作时产生的读视图，在该事务执行快照读的那一刻，会生成数据库系统当前的一个快照，记录并维护着当前活跃事务的ID(当每个事务开启时，都会被分配一个ID，这个ID是自增的，所以事务越新，ID越大)，但不包括该事务本身的事务ID  
Read View主要是用来做可见性判断的，即当前事务执行快照读的时候，对该记录创建一个Read View读视图，把它比作条件来判断当前事务能够看到哪个版本的数据，即可能是当前最新的数据，也有可能是该行记录的undo log中某一个版本的数据，这中间会遵循某种算法，下面会说明  
在说明之前，先简化一下Read View，可以把Read View理解成有三个全局属性  
1. trx_list  
一个数值列表，用来维护Read View生成时刻系统正在活跃的事务的ID，但是不包括生成Read View的事务的ID  
2. up_limit_id  
记录trx_list列表中最小的事务ID  
3. low_limit_id  
目前已出现过的最大事务ID+1  
**可见性遵循下面的算法**  
比较DB_TRX_ID<up_limit_id,如果成立，则说明当前事务能够看到DB_TRX_ID所在的记录，否则进入下一个判断  
接下来判断DB_TRX_ID>=low_limit_id,如果成立，则说明DB_TRX_ID所在的记录在Read View生成之后才出现，因此DB_TRX_ID所在的记录对当前事务不可见，否则进入下一个判断  
判断DB_TRX_ID是否在活跃事务当中，如果在，则说明Read View生成时刻，DB_TRX_ID的事务还没有提交，所以DB_TRX_ID修改的数据当前事务看不到；如果不在，说明Read View生成时刻，DB_TRX_ID的事务已经提交，所以DB_TRX_ID修改的数据当前事务可以看到  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/04/1599206807691-1599206807695.png)  
### 整体流程  
可以模拟一下整体的流程  
1. 有1，2，3，4四个事务，事务2执行了快照读，此时还有事务1和事务3处在活跃中，事务4在事务2执行快照读之前已经提交了修改  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/04/1599209133932-1599209133934.png)    
此时要判断事务2的快照读能够读到哪个事务做出的修改，因此trx_list的值为1、3，up_limit_id为1，low_limit_id为5，即已出现的最大事务ID+1  
此时该行记录和undo log为下图所示  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/04/1599207411857-1599207411864.png)  
因此事务2快照读时DB_TRX_ID字段记录的事务ID为4，下面开始判断  
首先先用快照读时的DB_TRX_ID字段记录的事务ID 4去和Read View中的up_limit_id 1去比较，发现4>1,不满足DB_TRX_ID<up_limit_id,进入下一个判断；接着用DB_TRX_id 4和Read View中low_limit_id 5去比较，发现4<5,进入下一个判断；发现4不在Read View的trx_list中，说明DB_TRX_ID字段为4的记录可以被事务2读取到  
2. 有1，2，3，4四个事务，事务2执行了快照读，此时还有事务1和事务3处于活跃状态中，事务4在事务2执行快照读之前提交了修改，事务1在事务4修改之后在事务2执行快照读之前进行了修改，但并未提交  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/04/1599209065965-1599209065967.png)
此时要判断事务2的快照读能够读到哪个事务做出的修改，因此trx_list的值为1、3，up_limit_id为1，low_limit_id为5，即已出现的最大事务ID+1  
此时该行记录和undo log为下图所示  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/04/1599208471244-1599208471246.png)  
因此事务2执行快照读时DB_TRX_ID字段记录的事务ID为1，下面开始判断  
首先先用快照读时的DB_TRX_ID字段记录的事务ID 1去和Read View中的up_limit_id 1去比较，发现1=1,不满足DB_TRX_ID<up_limit_id，进入下一个判断；接着用DB_TRX_ID 1和Read View中low_limit_id 5去比较，发现1<5，进入下一个判断；发现1在Read View的trx_list说明，说明DB_TRX_ID字段为1的事务还在活跃中，该事务的修改对当前事务不可见，所以沿着链表去寻找下一条记录，由图可以看出这条记录就是DB_TRX_ID为4的记录
然后先用DB_TRX_ID字段记录的事务ID 4去和Read View中的up_limit_id 1去比较，发现4>1,不满足DB_TRX_ID<up_limit_id,进入下一个判断；接着用DB_TRX_id 4和Read View中low_limit_id 5去比较，发现4<5,进入下一个判断；发现4不在Read View的trx_list中，说明DB_TRX_ID字段为4的记录可以被事务2读取到  
## MVCC相关问题  
### RR是如何在RC级别的基础上解决不可重复读的？  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/04/1599209767642-1599209767643.png)  
在上表的顺序下，事务B在事务A提交之后的快照读是旧版本的数据，当前读是新版本的数据  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/04/1599209908898-1599209908899.png)  
表2中快照读和当前读都是新版本的数据，这是为什么呢？  
**表1和表2中唯一的区别就在于首次快照读出现的地方。表1快照读出现在修改提交之前，表2快照读出现在修改提交之后。所以知道事务中快照读的结果是非常依赖该事务首次快照读出现的地方**  
### RC和RR隔离级别下，InnoDB快照读由什么不同  
正是Read View生成时机的不同，从而造成RC、RR级别下快照读的结果不同  
1. 在RR隔离级别下某个事务的对某条记录的第一次快照读会创建一个快照以及Read View，将当前系统活跃的其他事务记录起来，此后在调用快照读的时候，还是使用同一个Read View，所以只要当前事务在其他事务提交更新之前使用过快照读，那么之后的快照读使用的都是同一个Reda View，所以对之后的修改不可见  
2. 即在RR隔离级别下，快照读生成Read View时，Read View会记录此时所有其他活动事务的快照，这些事务的修改对当前事务都是不可见的；而早于Read View创建之前的事务所做的修改均是可见的  
3. 在RC隔离级别下，事务中，每次快照读都会生成一个新的Red View，这就是我们在RC隔离级别下的事务中可以看到别的事务提交的更新的原因  
**总之，在RC隔离级别下，是每个快照读都会生成并获取最新的Red View；而在RR隔离级别下，同一个事务的第一次快照读会创建一个Read View，之后的快照读都是获取这一个Read View**  