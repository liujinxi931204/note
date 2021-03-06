## YAML
### 简介  
YAML语言的设计目标，就是方便人类读写。它实质上是一种通用的数据串行化格式  
它的基本语法规则如下：  
*** 
+ 大小写敏感  
+ 使用缩进表示层级关系  
+ 缩进时不允许使用Tab键，只允许使用空格  
+ 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可  
*#* 表示注释，从这个字符一直到行尾，都会被解析器忽略  
### 数据结构  
#### 对象  
对象使用一组键值对表示，使用冒号结构  
```yml
animal : pets
hash : {name : Steve, foo : bar }
```  
#### 数组  
一组连词先开头的行，构成一个数组：  
```yml
- cat
- dog
- goldfish
```  
数据结构的子成员是一个数组，则可以在该项下面缩进一个空格  
```yml
-
 - cat
 - dog
 - goldfish
```  
数组也可以采用行内表示法  
```yml
{animals : ['cat', 'dog']}
```
#### 字符串  
正常情况下字符串不用写引号，字符串内有空格或者特殊字符时需要加引号  
```yml
str : 这是一行字符串  
str : '内容 : 字符串'    
```  
#### null
在YAML语言中使用~表示null  
```yml
parent : ~
```  
#### 复合结构  
对象和数组可以结合使用，形成复合结构  
```yml
languages : 
 - Ruby
 - Perl
 - Python
websites :  
 YAML : yaml.org
 Ruby : ruby.org
 Python : python.org
 Perl : use.per.org 
```