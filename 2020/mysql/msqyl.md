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
MySQL视图(view)是一种虚拟的表，同真实的表一样，视图也有行列构成，但视图并不存在在数据库中。行和列的数据来源于定义视图的查询所使用的表，并且还是在使用视图时动态的生成  

## MySQL事务  
DDL语言无法回滚，设计事务的时候不应该包含DDL语言  


