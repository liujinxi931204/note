# awk的程序结构如下：  

```shell
awk pattern {actuion}
```
awk的基本操作就是由输入行组成的序列中，陆续的扫描每一行，进行pattern匹配，匹配成功则会执行action中的动作。  

# awk的内建变量： 

NF表示当前行的字段数，所以$NF可以表示这行最后一个字段  

NR表示截至到目前读取的行数 

# awk常用内建变量：  

![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/18/1589767002974-1589767002976.png)
# awk的内建函数： 

length(s)返回字符串s的长度  

index(s,t)返回字符串t在s中第一次出现的次数，如果不存在，则返回0  

split(s,a)用FS将s分割到数组a中，返回字段的个数  

gsub(r,s)将$0中出现的r全部替换为s，返回替换的次数  

gsub(r,s,t)将字符串t中出现的r全部替换为s，返回替换的次数  

substr(s,p)返回字符串s中从p位置开始的所有的后缀  

substr(s,p,n)返回字符串中从p位置开始长度为n的后缀 


# awk常用内建函数：  

![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/18/1589767124676-1589767124678.png)
# 正则表达式：  

![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/18/1589767456222-1589767456225.png)

