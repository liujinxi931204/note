# MySQL的事务  
当多个用户同时访问同一数据时，一个用户在更改数据的过程中可能有其他的用户同时发起更改请求，为保证数据的一致性状态，MySQL引入了事务  
事务可以将一系列的数据操作捆绑成一个整体进行统一管理，如果某一事务执行成功，则改事务中进行的所有数据更改均会成功，成为数据库中的永久组成部分。如果事务执行时遇到错误，则就必须取消或回滚。取消或回滚后，数据将全部恢复到操作之前的状态，所有数据的更改均被清除  
## 事务的四个特性  
事务具有四个特性，即原子性(Atomicity)、一致性(Consistency)、隔离性(Isolation)、持久性(Durability)，这四个特性通常成为ACID  
### 原子性  
事务是一个完整的操作。事务的各元素是不可分的(原子的)。事务中的所有元素必须作为一个整体提交或回滚。如果事务中的任何元素失败，则整个事务将失败  
### 一致性  
当事务完成时，数据必须处于一致的状态。也就是说，在事务开始前，数据库中的数据处于一致状态。在正在进行的事务中，数据可能处于不一致的状态，如数据可能有部分被修改。然而，当事务成功完成时，数据必须再次回到已知的一致状态。通过事务对数据所做的修改不能损坏数据，或者说事务不能使数据处于不稳定的状态  
### 隔离性  
对数据进行修改的所有的并发事务是彼此隔离的，这表明事务必须是独立的。它不应该以任何方式依赖或影响其他事务。修改数据的事务可以在另一个使用相同数据的事务开始之前访问这些数据，或者在另一个使用相同数据的事务结束后访问这些数据  
### 持久性  
事务的持久性指不管系统是否发生了故障，事务处理的结果都是永久的。  
一个事务成功完成之后，他对数据库所作的更改是永久性的，即使系统出现故障也是如此。也就是说，一旦事务被提交，事务对数据所做的任何变动就永久地保存在数据库中  

事务的ACID原则保证了一个事务或者成功提交，或者失败回滚，二者必居其一。因此它对事务的修改具有可恢复性。即当事务失败时，它对数据的修改都会恢复到该事务执行前的状态  
## MySQL执行事务的语法和流程  
MySQL提供了多种存储引擎来支持事务，有InnoDB和BDB。其中，InnoDB存储引擎事务主要通过UNDO日志和REDO日志实现，MyISAM存储引擎不支持事务  
为了维护MySQL服务器，经常需要在MySQL数据库中进行日志操作：  
1. UNDO日志：复制事务执行前的数据，用于在事务发生异常时回滚  
2. REDO日志：记录在事务执行中，每条对数据进行更新的操作，当事务提交时，该内容将被刷新到磁盘  

默认设置下，每条SQL语句就是一个事务，即执行SQL语句后自动提交。为了达到将几个操作做为一个整体的目的，需要使用BEGIN或START TRANSACTION开启一个事务或者禁止当前会话的自动提交  
### 开始事务  
`BEGIN;`  
或  
`START TRANSACTION;`  
这个语句显示地标记一个事务地起始点  
### 提交事务  
MySQL使用下面的语句来提交事务  
`COMMIT;`  
commit表示提交事务，具体地说，就是将事务中所有对数据库的更新都写到磁盘上的物理数据库中，事务正常结束  
一旦执行了该命令，将不能回滚事务  
### 回滚事务  
MySQL使用下面的语句回滚事务  
`ROLLBACK`  
ROLLBACK表示撤销事务，即在事务运行过程中发生了某种故障，食物不能继续执行，系统将事务中对数据库的所有已完成的操作全部撤销，回滚到事务开始时的状态。这里的操作是指对数据的更新操作  
## 注意事项  
MySQL事务是一项非常消耗资源的功能  
### 1). 事务尽可能简短  
事务的开启到结束会在数据库管理系统里保留大量资源，以保证事务的原子性、一致性、隔离性和持久性。如果在多用户的系统中，较大的事务将会占用大量资源  
### 2). 事务访问中的数据量尽量最小  
当并发执行事务处理时，事务操作的数据量越少，事物之间对相同数据的操作就越少  
### 3). 查询数据时尽量不用使用事务  
对数据进行浏览查询操作并不会更新数据库的数据，因此应尽量不使用事务查询数据，避免过多的占用系统资源  
### 4). 在事务处理过程中尽量不要出现等待用户输入的操作  
在处理事务的过程中，如果需要等待用户输入，事务会长时间地占用资源，有可能造成系统阻塞  
## MySQL设置事务自动提交(开启和关闭)  
MySQL默认开启事务自动提交模式，即除非显示的开启事务(BEGIN或START TRANSACTION),否则每条SQL语句都会被当作一个单独的事务自动提交  

在MySQL中，可以通过show variables 语句查看事务提交模式  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/08/03/1596444835665-1596444835767.png)   
在MySQL中，可以使用set autocommit语句设置事务的自动提交模式，语法格式如下：  
`set autocommit=0|1|ON|OFF`  
对取值的说明：  
值为0和值为OFF：关闭自动提交。如果关闭自动提交，用户将会一直处于某个事务当中，只有提交或回滚后才会结束当前事务，重新开始一个新的事务  
值为1和值为ON：开启自动提交。如果开启自动提交，则没执行一条SQL语句，事务都会提交一次  
## MySQL事务隔离级别详解  

如果事务没有隔离级别，就容易出现脏读、不可重复读和幻读  
### 1). 脏读  
脏读指一个事务正在访问数据，并且对数据进行了修改，但是这种修改还没有提交到数据库中，这时，另一个事务也访问这个数据，然后使用了这个数据  

### 2). 不可重复读  
不可重复读是指在一个事务内，多次读取同一个数据  
在这个事务还没有结束时，另一个事务也访问了该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的数据可能时不一样的。这样在一个事务内两次读到的数据是不一样的，因此称为不可重复读  
### 3). 幻读  
幻读是，第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行，同时第二个事务也修改这个表中的数据行，这种修改时是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行  

MySQL定义了4种事务隔离级别来解决以上问题  
1. 读未提交(READ UNCOMMITED)  
2. 读已提交(READ COMMITED)  
3. 可重复读(REPEATABLE READ)  
4. 串行化(SERIALIZABLE)  

MySQL事务隔离级别可能产生的问题如下表所示  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/08/03/1596446278513-1596446278524.png)  
查看当前的隔离级别  
执行下面的语句  
`show variables like '%isolation%';`  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/08/03/1596447769119-1596447769121.png)  
设置隔离级别  
执行下面的语句  
`set session|global transaction isolation level read uncommited`  
global是指全局的  
seesion是指当前session  

还可以修改mysql配置文件mysql.ini   
```shell
[mysqld]  
transaction-isolation = REPEATABLE-READ
```

### 1. 读未提交  
读未提交就是可以读到未提交的内容  
如果一个事务读取到了另一个未提交事务修改过的数据，那么这种隔离级别就是读未提交  
在该隔离级别下，所有事务都可以看到其他未提交事务的执行过程。它的性能与其他隔离级别相比没有高多少，一般情况下，该隔离级别很少使用  
### 2. 读提交  
如果一个事务只能读取到另一个已提交事务修改过的数据，并且其他事务每对该数据进行一次修改并提交后，该事务都能查询到最新的值，那么这种隔离级别称为都提交  
改隔离级别满足了隔离的简单定义：一个事务从开始到提交所做的任何改变都是不可见的，事务只能读取到已提交的事务所做的改变  
### 3. 可重复读  
可重复读是MySQL的默认事务隔离级别，它能确保同一事务的多个实例在并发读取数据时，会看到同样的数据行。在该隔离级别下，可以解决不可重复读的问题，但是不能解决幻读的问题  
### 4. 串行化  
如果一个事务先根据某些条件查询出一些记录，之后另一个事务又向表中插入了符合这些条件的记录，原先的事务再次对该条件查询时，能把另一个事务插入的记录也读出来，那么这种隔离级别称为串行化  
SIRIALIZABLE是最高的隔离级别，主要通过强制事务排序来解决幻读问题。但是该隔离级别下，事务的并发率特别低，且性能开销也最大，一般不推荐使用  

在实现上，数据库里面会创建一个视图，访问的时候以视图的逻辑结果为准。在"可重复读"隔离级别下，这个视图是在事务启动时创建的，整个事务存在期间都用这个视图。在"读提交"隔离级别下，这个视图时在每个SQL语句开始执行的时候创建的。这里需要注意的时,"读未提交"隔离模式下直接返回记录上的最新值，没有视图概念；而"串行化"隔离级别直接用加锁的方式来避免并行访问  









 




