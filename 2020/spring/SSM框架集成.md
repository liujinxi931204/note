### SSM框架集成  
SSM框架整合指的是将Spring+SpringMVC+Mybatis框架进行整合  
Spring MVC负责实现MVC设计模式，Mybatis负责持久层，Spring 负责管理Spring MVC和Mybatis相关对象和依赖注入    
#### IDEA创建项目  
IDEA创建Maven web工程，pom.xml中添加相关依赖  
```xml
 <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
<!--    添加spring相关依赖-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
      <version>5.2.8.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>5.2.8.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.2.8.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-expression</artifactId>
      <version>5.2.8.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aop</artifactId>
      <version>5.2.8.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aspects</artifactId>
      <version>5.2.8.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>5.2.8.RELEASE</version>
    </dependency>
<!--    spring-jdbc相关依赖-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>5.2.8.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>5.2.8.RELEASE</version>
    </dependency>
<!--    spring-web相关依赖-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>5.2.8.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.2.8.RELEASE</version>
    </dependency>
<!--    mybatis相关依赖-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.3</version>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>2.0.3</version>
    </dependency>

<!--    mysql驱动-->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.47</version>
    </dependency>
<!--    数据库连接池-->
    <dependency>
      <groupId>com.mchange</groupId>
      <artifactId>c3p0</artifactId>
      <version>0.9.5.2</version>
    </dependency>
<!--     JSTL-->
    <dependency>
      <groupId>jstl</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
    </dependency>
<!--     Servlet api-->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>4.0.1</version>
      <scope>provided</scope>
    </dependency>
  </dependencies>
```  
如果源代码中还有*.xml、*.propertis、*.tld等配置文件，还需要再pom.xml文件中配置resources标签  
```xml
<build>
    <resources>
      <resource>
        <directory>src/main/resources</directory>
      </resource>
      <resource>
        <directory>src/main/java</directory>
        <includes>
          <include>**/*.xml</include>
          <include>**/*.propertis</include>
          <include>**/*.tld</include>
        </includes>
        <filtering>false</filtering>
      </resource>
    </resources>
  </build>
```  
#### 配置web.xml  
在web.xml中配置Spring MVC、Spring、字符编码过滤器、加载静态资源  
##### 配置web.xml  
```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0">
  <display-name>Archetype Created Web Application</display-name>

<!--  配置spring的监听器，默认只加载WEB-INF目录下的applicationContext.xml配置文件-->
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
<!--  web服务器启动时加载src/resources/下的applicationContext.xml配置文件-->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
  </context-param>

<!--  配置Spring MVC的前端控制器，为前端控制器指定配置文件为src/resources/spring-mvc.xml
      设置前端控制器拦截所有的url请求，并在web服务器启动的时候就加载DispatcherServlet
-->
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>

<!--  配置字符编码过滤器-->
  <filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
      <param-name>forceEncoding</param-name>
      <param-value>true</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>


</web-app>

```  
#### 配置Spring的配置文件，applicationContext.xml  
配置applicationContext.xml,主要需要配置注解扫描、数据库连接池、SqlSessionFactory bean对象、事务管理器和基于注解的声明式事务  
**数据库配置文件**  
```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://10.160.58.128:3306/user_db?charsetEncoding=utf-8
jdbc.username=root
jdbc.password=123456
```  
**Spring的配置文件applicationContext.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
">

<!--配置注解扫描-->
    <context:component-scan base-package="com.sogou.service,com.sogou.pojo"/>
<!--    使用db.properties数据库配置文件，引入配置文件-->
    <context:property-placeholder location="classpath:db.properties"/>

<!--    数据库连接池-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

<!--    配置SqlSessionFactory对象-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
<!--        注入数据库连接池-->
        <property name="dataSource" ref="dataSource"/>
<!--        扫描sql配置文件：mapper需要的xml文件
            这里mapper映射的xml文件路径如果和mapper.java定义在同一目录下，可以不用设置
            如果mapper映射的xml文件路径和mapper.java没有定义在同一目录下，则需要设置
<property name="mapperLocations" value="/com/sogou/mapper/*.xml"/>

-->
        
    </bean>

<!--    扫描dao接口包，动态实现dao接口，注入到spring容器-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
<!--        注入sqlSessionFactory-->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
<!--        给出需要扫描dao接口包-->
        <property name="basePackage" value="com.sogou.mapper"/>
    </bean>

<!--    配置事务管理器-->
    <bean id="dataSourceTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
<!--        注入数据连接池-->
        <property name="dataSource" ref="dataSource"/>
    </bean>

<!--    配置基于注解的声明式事务-->
    <tx:annotation-driven transaction-manager="dataSourceTransactionManager"/>

</beans>
```  
#### 配置Spring MVC的配置文件
**Spring MVC的DispatcherServlet配置文件spring-mvc.xml**  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd">

<!--    扫描controller相关的注解-->
    <context:component-scan base-package="com.sogou.controller"/>
<!--    开启Spring MVC注解模式-->
    <mvc:annotation-driven/>

<!--    配置访问静态资源，这里设置访问jQuery-->
    <mvc:resources mapping="/js/**" location="/js/"/>
<!--    配置访问静态资源，这里设置访问jsp-->
    <mvc:resources mapping="/jsp/**" location="/jsp/"/>

<!--    配置jsp，显示ViewResolver-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

</beans>
```  
**整合SSM，不再需要mybatis-config.xml核心配置文件**  
#### 对应实现类  
**Cateogry**  
```java
package com.sogou.pojo;

import org.springframework.stereotype.Component;


/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-27 11:34
 **/

@Component("category")
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
**Product**  
```java
package com.sogou.pojo;

import org.springframework.stereotype.Component;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-27 11:35
 **/

@Component("product")
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
#### 接口和对应的xml映射文件  
**getCategoryMapper**
```java
package com.sogou.mapper;

import com.sogou.pojo.Category;
import org.apache.ibatis.annotations.Param;

import java.util.List;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-27 11:37
 **/
@Repository("categoryMapper")
public interface getCategoryMapper {

    List<Category> findAllCategory();
    Category findCategoryById(@Param("id")int id);
}
```  
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sogou.mapper.getCategoryMapper">
    <select id="findAllCategory" resultType="com.sogou.pojo.Category">
    select * from category
  </select>

    <select id="findCategoryById" resultType="com.sogou.pojo.Category" parameterType="int">
        select * from category where id = ${id}
    </select>
</mapper>
```  
**getProductMapper**  
```java
package com.sogou.mapper;

import com.sogou.pojo.Product;
import org.apache.ibatis.annotations.Param;

import java.util.List;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-27 14:10
 **/
@Repository("productMapper")
public interface getProductMapper {

    List<Product> findAllProduct();
    Product findProductById(@Param("id")int id);
}

```  
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sogou.mapper.getProductMapper">
    <select id="findAllProduct" resultType="com.sogou.pojo.Product">
    select * from product
  </select>

    <select id="findProductById" resultType="com.sogou.pojo.Product" parameterType="int">
        select * from product where id = ${id}
    </select>
</mapper>
```  
#### 定义service接口和impl接口实现类  
**getCategoryDao**
```java
package com.sogou.service;

import com.sogou.pojo.Category;


import java.util.List;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-27 15:36
 **/
public interface getCategoryDao {

    public List<Category> getAllCategory();
    public Category getCategoryById(int id);


}
```  
**getCategoryImpl**
```xml
package com.sogou.impl;

import com.sogou.mapper.getCategoryMapper;
import com.sogou.pojo.Category;
import com.sogou.service.getCategoryDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-27 15:37
 **/
@Service("getCategory")
public class getCategoryImpl implements getCategoryDao {

    @Qualifier("categoryMapper")
    @Autowired
    private getCategoryMapper getCategoryMapper;

    @Override
    public List<Category> getAllCategory() {
        return getCategoryMapper.findAllCategory();
    }

    @Override
    public Category getCategoryById(int id) {
        return getCategoryMapper.findCategoryById(id);
    }
}

```
**getProductDao**
```xml
package com.sogou.service;

import com.sogou.pojo.Product;

import java.util.List;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-27 17:46
 **/
public interface getProductDao {

    public List<Product> getAllProduct();
    public Product getProductById(int id);

}

```
**getProductImpl**  
```xml
package com.sogou.impl;

import com.sogou.mapper.getProductMapper;
import com.sogou.pojo.Product;
import com.sogou.service.getProductDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-27 17:47
 **/

@Service("getProduct")
public class getProductImpl implements getProductDao {


    @Autowired
    @Qualifier("productMapper")
    private getProductMapper getProductMapper;

    @Override
    public List<Product> getAllProduct() {
        return getProductMapper.findAllProduct();
    }

    @Override
    public Product getProductById(int id) {
        return getProductMapper.findProductById(id);
    }
}
```  
####定义Spring MVC的控制器  
**categoryController**  
```java
package com.sogou.controller;

import com.sogou.impl.getCategoryImpl;
import com.sogou.pojo.Category;
import com.sogou.service.getCategoryDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-27 15:07
 **/

@Controller
@RequestMapping("/Category")
public class CategoryController {

    @Qualifier("getCategory")
    @Autowired
    private getCategoryImpl getCategoryImpl;

    @RequestMapping("/getCategory/{id}")
    public String getCategoryById(@PathVariable("id")int id,Model model){
        Category category = getCategoryImpl.getCategoryById(id);
        model.addAttribute("id",category.getId());
        model.addAttribute("name",category.getName());
        return "showCategory";
    }
}

```
**productController**  
```java
package com.sogou.controller;

import com.sogou.impl.getProductImpl;
import com.sogou.pojo.Product;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-27 17:50
 **/

@Controller
@RequestMapping("/Product")
public class ProductController {

    @Autowired
    @Qualifier("getProduct")
    private getProductImpl getProductImpl;


    @RequestMapping("/getProduct/{id}")
    public String getProductById(@PathVariable("id")int id, Model model){

        Product product = getProductImpl.getProductById(id);
        model.addAttribute("id",product.getId());
        model.addAttribute("name",product.getName());
        model.addAttribute("price",product.getPrice());
        model.addAttribute("cid",product.getCid());
        return "showProduct";
    }
}

```  






