awk的程序结构如下：
awk pattern {action}
awk的基本操作就是由输入行组成的序列中，陆续的扫描每一行，进行pattern匹配，匹配成功则会执行action中的动作。
awk的内建变量：
NF表示当前行的字段数，所以$NF可以表示这行最后一个字段
NR表示截至到目前读取的行数
awk的内建函数：



