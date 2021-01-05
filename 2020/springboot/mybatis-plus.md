## 简介  
Mybatis-Plus，简称MP，是一个Mybatis的增强工具，在Mybatis的基础上只做增强不做改变，为简化开发、提高效率而生  
### 支持数据库  
任何能使用mybatis进行crud，并且支持标准sql的数据库  
### 框架结构  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2021/01/05/1609810091271-1609810091331.png)  
### 安装  
```xml
<!--        引入mybatis-plus的starter-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.4.1</version>
        </dependency>
```
### 数据准备  
```sql
drop table if exists user;
CREATE TABLE `user` (
  `id` int(11) NOT NULL,
  `name` varchar(30) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `email` varchar(30) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8


drop table if exists book;
CREATE TABLE `book` (
  `book_id` int(10) NOT NULL AUTO_INCREMENT,
  `book_name` varchar(400) DEFAULT NULL,
  `price` float DEFAULT NULL,
  `content` text,
  PRIMARY KEY (`book_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```  
### Mapper CRUD接口 
**说明**  
通用CURD封装BaseMapper接口，为Mybatis-Plus启动时自动解析实体表关系映射转化为Mybatis内部对象注入容器  
BaseMapper接口的全限定名为`com.baomidou.mybatisplus.core.mapper.BaseMapper<T>`，该接口提供了插入、修改、删除和查询接口  
如下  
#### select 查询  
```java

```

















