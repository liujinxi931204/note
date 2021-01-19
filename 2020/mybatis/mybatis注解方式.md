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
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/10/19/1603092864744-1603092864746.png)  
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
#### 多对一的查询实现  
Category类的实现,这里的List是为了实现一对多的查询，注解实现sql语句的时候会用到  
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
package com.sogou.pojo;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-19 10:50
 **/
public class Product {
    private int id;
    private String name;
    private float price;
    private int cid;
    

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

    public float getPrice() {
        return price;
    }

    public void setPrice(float price) {
        this.price = price;
    }

    public int getCid() {
        return cid;
    }

    public void setCid(int cid) {
        this.cid = cid;
    }


    @Override
    public String toString() {
        return "Product{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", price=" + price +
                ", cid=" + cid +
                '}';
    }
}

```
接下来实现categroyMapper接口和productMapper接口，注意一定是接口，不能是实现类  
```java
package com.sogou.dao;

import com.sogou.pojo.Category;
import org.apache.ibatis.annotations.*;

import javax.naming.Name;
import java.util.Calendar;
import java.util.List;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-19 10:52
 **/
public interface categoryMapper {

    @Select("select * from category")
    List<Category> findAllCategory();

    @Select("select id,name from category where id=#{id}")
    Category findCategoryById(@Param("id") int id);


    @Results(id = "categoryBean" ,value = {
            @Result(column = "id" ,property = "id",id = true),
            @Result(column = "name",property = "name"),
            @Result(property = "productList",column = "id",
            many = @Many(select = "com.sogou.dao.productMapper.findProductByCid"))
    }
    )
    @Select("select id,name from category")
    List<Category> findWholeCategory();
}
```
```java
package com.sogou.dao;

import com.sogou.pojo.Product;
import org.apache.ibatis.annotations.*;
import org.apache.ibatis.javassist.util.proxy.ProxyObject;

import java.util.List;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-19 10:52
 **/
public interface productMapper {

    @Select("select id,name,price,cid from product")
    List<Product> findAllProduct();


    @Select("select id,name,price,cid from product as p where cid=#{cid}")
    Product findProductByCid(@Param("cid") int cid);


    @Insert("insert into product (name,price,cid) values(#{product.name},#{product.price},#{product.cid})")
    //返回自增主键
    @Options(useGeneratedKeys = true,keyProperty = "id")
    int addProduct(@Param("product")Product product);

    @Delete("delete from product where id=#{id}")
    int removeProduct(@Param("id") int id);

}

```
在这两个接口中，使用@select、@insert、@update、@delete这四个注解实现select、insert、update、delete语句  
因为这里使用的是注解，因此没有写xml配置文件  
+ @Results注解对应的是xml配置文件中的resultMap的实现，id的意义与配置文件中id的意义相同，不过在mybatis 3.3.0之前不支持，也就是说在该版本之前的@Results是不能复用的，每一次都需要重新写一遍  
+ @Result(column = "id" ,property = "id",id = true)就是resultMap中的<id/>标签  
```java
@Result(property = "productList",column = "id",
            many = @Many(select = "com.sogou.dao.productMapper.findProductByCid"))
```
这里用来实现多对一的查询，property属性是Category类中对应List的属性名，column则是传递给下面这个方法的参数  
many=@Many(select="com.sogou.dao.productMapper.findProductByCid") 用来是调用com.sogou.dao.productMapper这个接口中的findProductByCid的方法，并且将columu中的参数传递给它，这里可以看到一findProductByCid  
```java
@Select("select id,name,price,cid from product as p where cid=#{cid}")
    Product findProductByCid(@Param("cid") int cid);
```
**从上面可以看出，其实这个多对一查询进行了两次select查询**  
这一点从日志中也可以看到  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/10/19/1603096970949-1603096970951.png)  
#### 一对一的查询实现  
需要对上面的类进行修改  
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


    @Override
    public String toString() {
        return "Category{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}

```
Product类的实现  
```Product
package com.sogou.pojo;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-19 10:50
 **/
public class Product {
    private int id;
    private String name;
    private float price;
    private int cid;
    private Category category;

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

    public float getPrice() {
        return price;
    }

    public void setPrice(float price) {
        this.price = price;
    }

    public int getCid() {
        return cid;
    }

    public void setCid(int cid) {
        this.cid = cid;
    }

    public Category getCategory() {
        return category;
    }

    public void setCategory(Category category) {
        this.category = category;
    }

    @Override
    public String toString() {
        return "Product{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", price=" + price +
                ", cid=" + cid +
                ", category=" + category +
                '}';
    }
}

```
categoryMapper和productMapper接口的实现  
```java
package com.sogou.dao;

import com.sogou.pojo.Category;
import org.apache.ibatis.annotations.*;

import javax.naming.Name;
import java.util.Calendar;
import java.util.List;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-19 10:52
 **/
public interface categoryMapper {

    @Select("select * from category")
    List<Category> findAllCategory();

    @Select("select id,name from category where id=#{id}")
    Category findCategoryById(@Param("id") int id);


    @Results(id = "categoryBean" ,value = {
            @Result(column = "id" ,property = "id",id = true),
            @Result(column = "name",property = "name"),
            @Result(property = "productList",column = "id",
            many = @Many(select = "com.sogou.dao.productMapper.findProductByCid"))
    }
    )
    @Select("select id,name from category")
    List<Category> findWholeCategory();

}

```
```java
package com.sogou.dao;

import com.sogou.pojo.Product;
import org.apache.ibatis.annotations.*;
import org.apache.ibatis.javassist.util.proxy.ProxyObject;

import java.util.List;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-19 10:52
 **/
public interface productMapper {

    @Select("select id,name,price,cid from product")
    List<Product> findAllProduct();


    @Select("select id,name,price,cid from product as p where cid=#{cid}")
    Product findProductByCid(@Param("cid") int cid);


    @Insert("insert into product (name,price,cid) values(#{product.name},#{product.price},#{product.cid})")
    //返回自增主键
    @Options(useGeneratedKeys = true,keyProperty = "id")
    int addProduct(@Param("product")Product product);



    @Delete("delete from product where id=#{id}")
    int removeProduct(@Param("id") int id);


    @Results(id = "productBean",value =
            {
                    @Result(column = "id",property = "id",id = true),
                    @Result(column = "name",property = "name"),
                    @Result(column = "price",property = "price"),
                    @Result(column = "cid",property = "cid"),
                    @Result(property = "category",column = "cid",
                            one = @One(select = "com.sogou.dao.categoryMapper.findCategoryById"))

            })
    @Select("select * from product as p where id=#{id}")
    Product getProductById(@Param("id")int id);
}

```
这里其余注解意义和上面的相同，只有  
`@Result(property = "category",column = "cid",
 one = @One(select = "com.sogou.dao.categoryMapper.findCategoryById"))`  
对应的是一对一的查询，与多对一的查询一样，也需要由两次查询才能完成  







  



