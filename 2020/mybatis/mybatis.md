## 简介  
### maven配置  
```xml
<dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.6</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.12</version>
        </dependency>
    </dependencies>
```  
使用mybatis需要mybatis、mysql-connector-java的jar包，使用maven可以直接在maven的pom.xml文件中配置完成  
项目的目录结构如下：  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/10/19/1603092864744-1603092864746.png)  
目录结构说明：  
+ dao目录：DAO层，主要是对数据库的操作  
+ pojo目录：POJO层，数据库的具体类的实现  
+ utils目录：工具层，一些mybatis中常用工具的封装  
+ resources目录：配置文件所在目录，包括数据配置文件、mybatis核心配置文件等  
+ test目录：junit测试  
#### utils常用工具类  
```java
package com.sogou.utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-11 20:32
 **/
public class mybatisUtils {

    private static SqlSessionFactory sqlSessionFactory;

    static{
        String resources="mybatis-config.xml";
        try {
            InputStream inputStream = Resources.getResourceAsStream(resources);
            sqlSessionFactory= new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession();
    }
}
```  
这个类主要作用是获取sqlSession，这是mybatis中的核心，sqlSession可以对数据库进行增删改查的操作  
#### mybatis  
  



