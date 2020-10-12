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
```

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
