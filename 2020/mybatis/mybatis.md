## 简介  
### maven配置  
```java
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.6</version>
    </dependency>
```  
### 持久层  
数据持久化  
+ 持久化就是将程序的数据在持久状态和瞬时状态转化的过程  
### 编写mybatis核心配置文件  
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<!--    mybatis核心配置文件的标签有顺序-->
<!--    properties引入外部配置文件，使用resource指定外部的配置文件-->

    <properties resource="db.properties"/>

<!--    enviornments指定当前的环节，一般有两个环境，一个用于测试，一个用于开发-->
<!--    通过default来指定使用哪一个环境，例如这里指定使用development环境-->
<!--    enviornment通过id来唯一标识环境-->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="category.xml"/>
    </mappers>
</configuration>
```  
### 数据配置文件db.properties
```java

```

### 编写mybatis的工具类，mybatisUtils.java
```java
package com.sogou;

//sqlSessionFactory---->sqlSession
public class mybatisUtils{
//获取sqlSessionFactory对象
    static{
        try{
            String resources = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resources);
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);    
        }
        catch (IOException e){
            e.printStackTrace();
        } 
    }
}





```
