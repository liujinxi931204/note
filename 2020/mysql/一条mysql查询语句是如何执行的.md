假设有一张表T，里面只有一个字段ID,当执行下面这条SQL语句时：  
`select * from T where ID=10；`  
我们可以肉眼看到输入只是一条SQL语句，返回的是查询结果，却不知道这条SQL语句在MySQL内部经历了哪些。如下是MySQL的基本架构示意图，从图中可以看到SQL语句在MySQL中各个功能模块的执行过程  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/07/27/1595839894222-1595839894224.png)  
大体来说，MySQL可以分为Server层和存储引擎两部分。Server层：包括连接器、分析器、查询缓存、优化器、执行器等。存储引擎：负责数据的存储和提取
