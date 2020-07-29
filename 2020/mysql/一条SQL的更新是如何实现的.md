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


