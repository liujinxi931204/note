![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/07/27/%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6%20(5)-1595839204079.jpg)  
上图是MySQL的基本结构示意图  

假设有一张表T，里面只有一个字段ID,当执行下面这条SQL语句时：  
```select * from T where ID=10；```  
我们可以肉眼看到输入只是一条SQL语句，返回的是查询结果，却不知道这条SQL语句在MySQL内部经历了哪些。
