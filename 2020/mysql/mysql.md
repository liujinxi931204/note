



![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/06/15/mysql7%E7%A7%8Djoin-1592204990834.png)  

## SQL主要包含4个部分  
### 数据定义语言(Data Definition Language,DDL)  
用来创建或删除数据库以及表等对象，主要包含以下几种命令  
DROP：删除数据库和表等对象  
CREATE：创建数据库和表等对象  
ALTER：修改数据库和表等对象  
TRUNCATE：删除当前表再建一个相同的  
### 数据操纵语言(Data Manipulation Language,DML) 
用来变更表中的记录的，主要包含以下几种命令   
SELECT：查询表中的数据  
INSERT:向表中插入数据  
UPDATE：更新表中的数据  
DELETE：删除表中的数据  
### 数据查询语言(Data Query Language,DQL)  
用来查询表中的数据，主要包含SELECT命令  
### 数据控制语言(Data Control Language,DCL)  
用来确认或者消除数据库中的数据进行的变更，除此之外，还可以对数据库中的用户设定权限  
GRANT：赋予用户操作权限  
REVOKE：取消用户操作权限  
COMMIT：确认对数据库中的数据进行变更  
ROLLBACK：取消对数据库中的数据进行变更 
## MySQL视图  
MySQL视图(view)是一种虚拟的表，同真实的表一样，视图也有行列构成，但视图并不存在在数据库中。行和列的数据来源于定义视图的查询所使用的表，并且还是在使用视图时动态的生成.因此，视图中的数据时依赖真是表中的数据的，如果真实表中的数据发生了改变，视图中的数据也会发生改变.mysql的视图不支持参数输入的功能，比较适合变化不是很大的操作  
视图不能索引，也不能有关联的触发器、默认值或规则  
视图不包含数据，所以每次使用视图的适合都必须执行查询中所需的任何一个检索操作。如果用多个连接和过滤条件创造了复杂的视图或嵌套类视图，都可能会发现系统的性能下降的十分严重  
ORDER BY字句可以使用在视图中，但是若该视图检索的SELECT语句也含有ORDER BY子句，则视图中的ORDER BY会被覆盖掉  
### 创建视图  
基本格式  
`CREATE VIEW <视图名> AS <SELECT 语句>`
说明  
<视图名>：指定视图的名称。该名称在数据库中必须是唯一的，不能和别的表或视图重名  
<SELECT语句>：指定创建视图的SELECT语句，可以用于查询多个基础表或源视图  
创建视图存在以下限制  
SELECT语句不能引用系统变量或用户变量  
SELECT语句不能包含FROM子语句中子查询  
SELECT语句不能引用预处理语句参数  
视图中不能引用临时表  
### 修改视图  
基本语法  
`ALTER <视图名> AS <SELECT 语句>`
视图是一个虚拟表，实际的数据来自于基本表，所以通过插入、删除和修改操作更新视图的数据，实质是在更新视图所引用的基本表的数据。因此在修改表时要符合基本的数据定义  
如果视图包含以下结构之一，就是不可更新的  
```shell
聚合函数COUNT() SUM() MAX() MIN()等  
DISTINCT关键字  
GROUP BY关键字  
HAVING关键字  
UNION 或UNION ALL运算符  
位于选择列表中的子查询  
FROM字句中的不可跟新视图或包含多个表  
WHERE字句中的子查询，引用FROM子句中的表  
ALGORTHIM选项为TEMPTABLE（使用临时表总会使视图称为不可更新的）
```
修改视图的名称可以先将视图删除，然后按照相同的定义语句进行视图的创建，并命名为新的视图的名称  
### 删除视图  
`DROP VIEW <视图1>[,<视图2>...]`  
## MySQL事务  
DDL语言无法回滚，设计事务的时候不应该包含DDL语言  
## MYSQL 索引  
索引是mysql数据库中最重要的对象之一，用于快速找出某个列中某一个特定值的行  
索引就是根据表中的一列或者若干列按照一定顺序建立的列值与记录行之间的对应关系，实质上是一张描述索引列的列值与原表中记录行之间一一对应关系的有序表  
在mysql中通常有两种方式访问数据库表的行数据  
### 顺序访问  
顺序访问是在表中实行全表扫描，从头到尾逐行遍历，直到无序的行数据中找到符合条件的目标数据。这种方式实现比较简单，但是当表中有大量数据的时候，效率非常低下  
### 索引访问  
索引访问是通过遍历索引来直接访问表中记录行的方式，使用这种方式的前提是对表建立一个索引，在列上建立了索引之后，查找数据可以直接根据该列上的索引找到对应的记录行的位置，从而快速地查找到数据。索引存储了指定列数据值地指针，根据指定的排序顺序对这些指针排序  
### 索引分类  
主要分为两类：
#### B树索引  
目前大部分额度索引都是采用B树索引来存储的。B树索引是一个典型的数据结构。其包含的组件主要有以下几个：  
1.叶子节点：包含的条目直接指向表里的行数据。叶子节点间彼此相连，一个叶子节点有一个指向下一个叶子节点的指针  
2.分支节点：包含的条目指向索引里其他的分支节点或者叶子节点  
3.根节点：一个B树索引只有一个根节点，实际上就是位于树的最顶端的分支节点  
基于这种树形数据结构，表中的每一行都会在索引位置上有一个对应值，因此在表中进行数据查询时，可以根据索引值一步一步定位到数据所在的行  
B树索引可以进行全键值、键值范围和键值前缀查询，也可以对查询结果进行order by排序，但B树索引必须遵循左边前缀原则。要考虑以下几点约束  
1.查询必须从索引的最左边的列开始  
2.查询不能跳过某一索引列，必须按照从左到右的顺序进行匹配  
3.存储引擎不能使用索引中范围条件右边的列  
#### 哈希索引  
mysql目前仅有MEMORY存储引擎和HEAP存储引擎支持这类索引，其中MEMORY引擎支持B树索引和哈希索引，默认是哈希索引  
哈希索引是根据索引列对应的哈希值的方法获取表的记录行，哈希索引最大的特点就是访问速度快，但也存在以下缺点  
1.mysql需要读取索引列的值来参与哈希计算，相对于B树索引来说，建立哈希索引会耗费更多的时间  
2.不能使用哈希索引排序  
3.哈希索引只支持等值比较 "=" "IN()" "<=>"  
4.哈希索引不支持键的部分匹配，因为在计算hash值得时候是通过整个索引值来计算的  
## 索引的使用原则和注意事项  
在经常需要搜索的列上建立索引，可以加快搜索的速度  
在作为主键的列上创建索引，强制该列的唯一性，并组织表中数据的排列结构  
在经常使用表连接的列上创建索引，这些列主要是一些外键，可以加快表连接的速度  
在经常需要根据范围进行搜索的列上创建索引，因为索引已经排序，所以其指定范围是连续的  
在经常需要排序的列上常见索引，因为索引已经排序，所以查询时可以利用索引的排序，加快排序查询  
在经常使用WHERE子句的列上创建索引，加快条件的判断速度  
## 索引的创建  
### 基本语法  
#### CREATE INDEX
可以使用CREATE INDEX语句在一个已有的表上创建索引，但该语句不能创建主键  
`CREATE <索引名> ON <表名>(<列名>[<长度>][DESC|ASC])`
索引名：指定索引名，一个表可以创建多个索引，但是每个索引在该表中的名称是唯一的   
表名：指定要创建索引的表名  
列名：指定要创建索引的列名，通常可以考虑将查询语句中在JOIN子句和WHERE子句里经常出现的列作为索引  
长度：可选项，指定使用列前length个字符来创建索引，使用列的一部分创建索引有利于减小索引文件的大小
ASC|DESC使用升序还是降序来排列  
#### CREATE TABLE  
索引也可以在建表的同时创建，在CREATE TABLE语句中添加以下语句  
`CONSTRAINT PRIMARY KEY [索引类型]（<列名>,...）`  
在CREATE TABLE语句中添加此语句，表示在创建新表的同时创建该表的主键  
`KEY|INDEX [<索引名>] [<索引类型>]（<列名>，...）`
在CREATE TABLE语句中添加此语句，表示在创建新表的同时创建该表的索引  
`UNIQUE [INDEX|KEY] [<索引名>] [索引类型] （<列名>，...）`
在CREATE TABLE语句中添加此语句，表示在创建新表的同时创建该表的唯一索引  
`FOREIGN KEY <索引名> <列名> `  
在CREATE TABLE语句中添加此语句，表示在创建新表的同时创建该表的外键  
#### ALTER TABLE  
语法格式  
`ADD INDEX [<索引名>] [<索引类型>] (<列名>，...)`  
在ALTER TABLE语句中添加此语法成分，表示在修改表的同时为该表添加索引  
`ADD PRIMARY KEY [<索引名>] （<列名>，...）`  
在ALTER TABLE语句中添加此语法成分，表示在修改表的同时为该表添加主键  
`ADD UNIQIE [KEY|INDEX] [<索引名>] [<索引类型>]（<列名>，...）`  
在ALTER TABLE语句中添加此语法成分，表示在修改表的同时为该表添加唯一性索引  
`ADD FOREIGN KEY [<索引名>] (<列名>,...)`  
在ALTER TABLE语句中添加此语法成分，表示在修改表的同时为该表添加外键  
### 查看索引  
`SHOW INDEX FROM <表名>[FROM <库名>]`  
## 存储过程  
存储过程是一组为了完成特定功能的SQL语句集合，使用存储过程的目的是将常用或复杂的工作预先用SQL语句写好并用一个指定名称存储起来，这个过程经编译和优化后存储在数据库服务器中，因此称为存储过程。当以后需要数据库提供与预定义好的存储过程功能相同的服务时，只需要"CALL存储过程名字"即可自动完成  
### 基本语法  
```shell
CREATE PROCEDURE <过程名> ([过程参数[,...]]) <过程体>
[过程参数[,...]] 格式
[IN|OUT|INOUT]<参数名><类型>
```
说明如下  
#### 过程名  
存储过程的名称，默认在当前数据库中创建，如果需要在特定的数据库中创建存储过程，则要在名称前面加上数据库的名称，即db_name.sp_name  
#### 过程参数  
其中<参数名>为参数名，<类型>为参数类型（可以是任意有效的MySQL数据类型）。当有多个参数时，参数列表中使用都好彼此分隔。存储过程可以没有参数，此时存储过程名称后面仍然需要一对括号  
MySQL存储工程支持三种类型的参数，即输入参数、输出参数、输入\输出参数，分别用IN、OUT、INOUT三个关键字标识。其中，输入参数可以传递给一个存储过程，输出参数用于存储过程需要返回一个操作结果的情形，而输入、输出参数既可以充当输入参数也可以充当输出参数  
#### 过程体  
存储过程的主体部分，包含在过程调用的时候必须执行的SQL语句，这个部分以关键字BEGIN开始，以关键字END结束。若存储过程体中只有一条SQL语句，则可以省略BEGIN-END标志  
在存储过程中有一个经常用到的命令DELIMITER命令，用来修改SQL语句结束符  
MySQL中可以通过SHOW PROCEDURE STATUS语句查看存储过程的状态  
`SHOW PROCEDURE STATUS LIKE "存储过程名"`  
`SHOW CREATE PROCEDURE 存储过程名`语句查看存储过程定义  
### 修改存储过程  
MySQL通过ALTER PROCEDURE来修改存储过程  
`ALTER PROCEDURE 存储过程名[特征...]`  
特征包括  
`CONTAINS SQL`表示子程序包含的SQL语句，但不包含读或写数据的语句  
`NO SQL`表示子程序中不包含SQL语句  
`READ SQL DATA`表示子程序中包含读数据的语句  
`MODIFIES SQL DATA`表示子程序中包含写数据的语句  
`SQL SECURITY {DEFINER|INVOKER}`指明谁有权限来执行  
`DEFINER`表示只有定义这自己才能够执行  
`INVOKER`表示调用者可以执行  
`COMMENT 'string'`表示注释信息  
修改存储过程只能修改存储过程的某些特征，如果需要修改语句或者名称，可以删除旧的写一个新的  
### 删除存储过程  
`DROP {PROCEDURE|FUNCTION}[IF EXISTS]<过程名>`  
过程名：指定要删除的存储过程名  
IF EXISTS:指定这个关键字，用于防止因删除不存在的存储过程而引发错误  
存储过程名后面没有参数列表，但是在删除之前需要确认该存储过程没有任何依赖，否则会导致与之关联的其他关联的存储过程失效  












































