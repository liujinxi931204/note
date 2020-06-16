## MySQL索引  
###  MySQL索引本质  
mysql索引的本质是数据结构。mysql在维护数据之外，还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向数据）,这样就可以可以在这些数据结构上实现高级查找算法。这种数据结构就是索引  
### 索引为什么使用B-Tree(B+Tree)  
一般来说，索引本身也很大，不可能全部存储在内存中，因此索引往往以文件的形式存储在磁盘上。这样的话，索引的查找过程就要产生磁盘IO，相对于内存存取，磁盘IO的耗时要高好几个数量级。换句话说，索引的结构组织要尽量减少查找过程中磁盘的IO次数。  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/06/16/1592295546811-1592295546890.png)  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/06/16/1592295586485-1592295586488.png)  
根据B-Tree的定义，可知检索一次最多需要访问h个节点。数据库系统的设计者巧妙地利用磁盘预读原理，讲一个节点地大小设为等于一个页的大小，这样每个节点只需要一次IO就可以完全载入。B-Tree中一次检索最多需要h-1次IO(根节点常驻内存)，时间复杂度未O(h)=O(logdN)。一般应用中，出度d是非常大的数字，通过超过100，因此h非常小，通常不超过3  
### MyISAM索引实现  
MyISAM使用B+Tree作为索引结构，叶节点的data域存放的是数据记录的地址。  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/06/16/1592297292999-1592297293007.png)  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/06/16/1592297470994-1592297470996.png)  
MyISAM中检索的算法首先按照B+Tree搜索算法搜索索引，如果指定的Key存在，则取出其data域的值，然后以data域保存的值未地址，读取相应的记录。MyISAM的索引方式叶叫做"非聚集索引"  
### InnoDB索引实现  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/06/16/1592298857263-1592298857265.png)  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/06/16/1592298874655-1592298874657.png)  
InnoDB也使用B+Tree作为索引结构，但具体的实现方式缺不相同  
1.InnoDB的数据文件本身就是索引文件，即InnoDB的叶子节点的data域保留着完整的数据行记录，这种索引也称为聚集索引。所以,InnoDB的数据文件本身要按照主键聚集，所以InnoDB要求表必须有主键(MyISAM)可以没有，如果没有显示指定，数据库会自动选择一个可以唯一标识数据记录的列作为主键，如果不存在，MySQL会自动生成一个隐含字段，这个字段的长度为6个字节，类型为长整型  
2.InnoDB的辅助索引的data域存储相应的记录主键的值而不是地址，换句话说，InnoDB的所有辅助索引都引用主键作为data域。聚集索引这种实现方式使得按照主键的搜索十分高效，但是辅助索引搜索需要检索两遍：首先检辅助索引获得主键，然后用主键到主索引中检索获得记录。
### 最左前缀原理与相关优化  
以上说的索引都是单一索引，实际上，MySQL中的索引可以以一定顺序引用多个列，这个索引叫做联合索引。  
以下是一个例子  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/06/16/1592299728720-1592299728722.png)  
从结果中可以看出titles表的主索引是一个联合索引(emp_no,title,from_date),还有一个辅助索引。下面主要分析索引PRIMARY的行为  
#### 全列匹配  
`explain select * from employees.titles where emp_no='10001' and title='Senior Engineer' and `





