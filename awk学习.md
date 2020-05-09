awk的程序结构如下<br>：
awk pattern {action}
awk的基本操作就是由输入行组成的序列中，陆续的扫描每一行，进行pattern匹配，匹配成功则会执行action中的动作。
awk的内建变量：
NF表示当前行的字段数，所以$NF可以表示这行最后一个字段
NR表示截至到目前读取的行数
awk的内建函数：
length(s)返回字符串s的长度
index(s,t)返回字符串t在s中第一次出现的次数，如果不存在，则返回0
split(s,a)用FS将s分割到数组a中，返回字段的个数



