假设创建这样一个表，这个表有一个主键ID和一个整型字段c：  
`create table T(ID primary keym,c int)；`  
如果要将ID=2这一行的值加1，SQL语句会这么写  
`update T set c=c+1 where ID=2;`  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/07/29/1596008975036-1596008975118.png)  
在这个表上有更新的时候，会把跟这表有关的查询缓存清空，所以一般不建议使用查询缓存  
与查询流程不同，更新流程还涉及两个日志redo log(重做日志)和bin log(归档日志)  
## 重要的日志模块  
### WAL  
WAL的全称是Write-Ahead Logging，它的关键点是先日志，再写磁盘。具体来说，当有一条记录需要更新的时候，InnoDB引擎就会先把记录写到redo log里面，并更新内存，这个时候就算更新完了。同时，InnoDB引擎会在适当的时候，将这个操作记录更新到磁盘里面，而这个更新往往是在系统比较空闲的时候  
## redo log
InnoDB的redo log的大小是固定的。从头开始写，写到末尾又回到开头循环写，如下面这个图所示  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/07/29/1596011619460-1596011619462.png)  
write pos是当前的记录位置，一边写一边后移，写到第3号文件末尾就回到0号文件的开头。check point是当前要擦除的位置，也是往后推移并且循环的，擦除记录前要把记录更新到数据文件。write pos和check point之间的位置就是可以记录更新的操作。如果write pos追上了check point这是就不能再执行更新，得停下来清理记录，然后再往前推动check point  
有了redo log，InnoDB就可以保证技术数据库发生异常重启，之前提交的记录都不会丢失，这个能力称为crash-safe  
## bin log  
MySQL整体来看，其实就是两个模块：一块是Server层，主要做的是MySQL功能层面的事情；另一块是引擎层，负责存储相关的具体事宜。redo log是InnoDB引擎特有的日志，而Server层也有自己的日志，称为bin log(归档日志)  
这两种日志有以下三点不同  
1.redo log是InnoDB特有的；bin log是Server层实现的，所有的引擎都可以使用  
2.redo log是物理日志，记录的是"在某个数据页上做了什么修改";bin log是逻辑日志，记录的是这个语句的原始逻辑，比如"给ID=2这一行的c字段加1"  
3.redo log是循环写的，空间固定会用完；bin log是可以追加写入的。追加写是指bin log文件写到一定的大小后会切换到写一个，并不会覆盖以前的日志  
再来看看执行器和InnoDB引擎在执行简单的update语句时的内部流程  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/07/29/1596013143217-1596013143221.png)  
1.执行器先找引擎取ID=2这一行。ID时主键，引擎直接用树搜索找到这一行。如果ID=2这一行所在的数据页本来就在内存中，就直接返回给执行器；如果不在内存中，需要线从磁盘中读入内存，然后再返回  
2.执行器拿到引擎给的行数据，把这个值加1，比如原来时N，现在得到N+1，得到新的一行的数据，再调用引擎接口写入这个行数据  
3.引擎将这个行数据更新到内存中，同时将这个更新操作记录到redo log中，此时redo log处于perpare状态；然后告知执行器执行完成了，随时可以提交事务  
4.执行器生成这个操作的bin log，并把bin log写入磁盘  
5.执行器调用引擎的提交事务接口，引擎把刚刚写入的redo log改成已提交的状态(commit)状态，完成更新  
上图中深色部分是在执行器中执行的，浅色部分是在InnoDB中执行的  

## 两段提交  
当需要恢复到指定的某一秒时，可以这么做  
首先，找到最近一次全量备份，从这个备份中恢复到临时库  
然后，从备份的时间点开始，将备份的bin log依次取出来，重放到需要恢复的那个时刻  

### 必要性  
如果不使用“两端提交”，那么数据库的状态就有可能和用它的日志恢复出来的库的状态不一致  
假设当前ID=2的行，字段c的值是0  
如果不采用"两端提交"的方式，日志的写法无非两种  
1.**先写redo log后写bin log**。假设，redo log写完，bin log还没有写的时候MySQL发生异常重启。由于redo log写完之后，系统即使崩溃，仍然能够把数据恢复回来，所以恢复这一行c的值是1  
但是由于bin log没写完就crash了，这时候bin log里面就没有记录这个语句。因此，之后备份日志的时候，存在来的bin log里面就没有这条语句  
如果需要用这个bin log来恢复临时库的话，由于这个语句的bin log丢失，这个临时库就会少这一次的更新，恢复出来的这一行c的值就是0，与原库的值不同  
2.**先写bin log后写redo log**。如果在bin log写完之后crash，由于redo log还没有写，崩溃恢复以后这个事务无效，所以这一行c的值是0。但是bin log里面已经记录了"把c从0更新为1"这个日志。所以，在之后用bin log来恢复的时候就多了一个事务出来，恢复出的这一行的c值就是1，与原库的值不同  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/07/29/1596013143217-1596013143221.png)  
### 两阶段提交发生时机  
1. 单独写一个update语句的时候，就默认提交事务，两阶段提交就发生在"提交阶段"  
2. 如果是begin-commit语句的序列，在执行commit这个语句的时候发生两阶段提交  

如果在图中A的地方，即写入redo log处于perpare阶段之后，写入bin log之前发生了crash，由于此时bin log还没有写，redo lod也没有提交，所以崩溃恢复的时候，这个事务会回滚。

如果在图中B的地方，即写入bin log之hi哦，还没有commit之前发生crash，需要分别处理
1.如果redo log里面的事务只有完整的perpare，则判断对应的