假设创建这样一个表，这个表有一个主键ID和一个整型字段c：  
`create table T(ID primary keym,c int)；`  
如果要将ID=2这一行的值加1，SQL语句会这么写  
`update T set c=c+1 where ID=2;`  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/07/29/1596008975036-1596008975118.png)  
在这个表上有更新的时候，会把跟这表有关的查询缓存清空，所以一般不建议使用查询缓存  
与查询流程不同，更新流程还涉及两个日志redo log(重做日志)和bin log(归档日志)