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
```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--核心配置文件-->
<configuration>
    <typeAliases>
        <package name="com.sogou"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://10.160.58.128:3306/user_db"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="Category.xml"/>
    </mappers>
</configuration>
```  
### 编写mybatis的工具类，mybatisUtils.java
```java
package com.sogou;

public class mybatisUtils{
//
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
