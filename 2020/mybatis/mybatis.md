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
#### mybatis核心配置文件  
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<!--    mybatis核心配置文件的标签有顺序-->
<!--    properties引入外部配置文件，使用resource指定外部的配置文件-->
    <properties resource="db.properties"/>
    
    <settings>
<!--        标准的日志工厂实现-->
        <setting name="logImpl" value="STDOUT_LOGGING"/>
<!--        使用log4j日志工厂，首先在maven的配置文件中导入logj的包-->
<!--        其次编写log4j的配置文件，log4j.perproties-->
<!--        <setting name="logImpl" value="LOG4J"/>-->
<!--        开启驼峰命名法，例如mysql数据库中的数据列名，create_time会自动转化为createTime-->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>

<!--    类型别名，使用alias后面的名称代替类型的全限定名-->
<!--    即，所有使用com.sogou.pojo.Category的地方都可以使用Category代替-->
    <typeAliases>
        <typeAlias type="com.sogou.pojo.Category" alias="Category"/>
        <typeAlias type="com.sogou.pojo.Product" alias="Product"/>
    </typeAliases>

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
<!--    映射器有几种形式
        一种是 resources="com/sogou/dao/categoryMapper.xml"配置文件的形式
        一种是class="com.sogou.dao.categoryMapper"全限定名的形式
        这里使用注解的方式实现sql语句，因此采用了全限定名的映射器配置      
-->
        <mapper class="com.sogou.dao.categoryMapper"/>
        <mapper class="com.sogou.dao.productMapper"/>
    </mappers>
</configuration>
```  
**mybatis核心配置文件的各个标签有严格的顺序**  
### 使用注解的方式实现sql语句  
Category类的实现  
```java
package com.sogou.pojo;

import java.util.List;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-19 10:49
 **/
public class Category {
    private int id;
    private String name;
    List<Product> productList;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<Product> getProductList() {
        return productList;
    }

    public void setProductList(List<Product> productList) {
        this.productList = productList;
    }

    @Override
    public String toString() {
        return "Category{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", productList=" + productList +
                '}';
    }
}
```  
Product类的实现  
```java

```

  



