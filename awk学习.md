# awk的程序结构如下：<br>
awk pattern {action}
awk的基本操作就是由输入行组成的序列中，陆续的扫描每一行，进行pattern匹配，匹配成功则会执行action中的动作。<br>
# awk的内建变量：<br>
NF表示当前行的字段数，所以$NF可以表示这行最后一个字段<br>
NR表示截至到目前读取的行数<br>
# awk的内建函数：<br>
length(s)返回字符串s的长度<br>
index(s,t)返回字符串t在s中第一次出现的次数，如果不存在，则返回0<br>
split(s,a)用FS将s分割到数组a中，返回字段的个数<br>
split(s,a,fs)用fs将s分到数组a中，返回字段的个数<br>
gsub(r,s)将$0中出现的r全部替换为s，返回替换的次数<br>
gsub(r,s,t)将字符串t中出现的r全部替换为s，返回替换的次数<br>
substr(s,p)返回字符串s中从p位置开始的所有的后缀<br>
substr(s,p,n)返回字符串中从p位置开始长度为n的后缀<br>






