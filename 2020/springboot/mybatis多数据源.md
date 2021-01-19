在某些场景下可能需要配置多个数据源，使用多个数据源(例如实现数据库读写分离)来缓解系统的压力等。这里主要是springboot+mybatis实现多数据源的配置  
### 系统目录  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/11/20/1605843230737-1605843230739.png)   
### 环境准备  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.sogou</groupId>
    <artifactId>bootdemo3</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>bootdemo3</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>11</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.4</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
<!--        引入druid依赖-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
            </resource>
        </resources>
    </build>
</project>
```
### 具体实现  
#### 多数据源配置  
首先编写application.properties配置文件，需要注意的是url一项需要写为  
```properties
spring.datasource.one.jdbcUrl=jdbc:mysql://10.160.58.128:3306/test_ljx
```
官方的解释为  
因为连接池的数据类型没有被公开，所引在您的自定义数据源的元数据中没有生成密钥，而且在IDE中没有完成(因为DataSource接口没有暴露属性)。另外，如果您碰巧在类路径上有Hikari，那么这个基本设置就不起作用了，因为Hikari没有url属性(但是确实有一个jdbcUrl属性)  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/11/20/1605841053254-1605841053256.png)  
**如果配置成了spring.datasource.one.url=jdbc:mysql://10.160.58.128:3306/test_ljx会报jdbcUrl is required with driverClassName.错误**
```properties
spring.datasource.one.jdbcUrl=jdbc:mysql://10.160.58.128:3306/test_ljx
spring.datasource.one.username=root
spring.datasource.one.password=123456
spring.datasource.one.driverClassName=com.mysql.cj.jdbc.Driver
spring.datasource.one.type=com.alibaba.druid.pool.DruidDataSource

spring.datasource.two.jdbcUrl=jdbc:mysql://10.160.58.128:3306/test_ljx
spring.datasource.two.username=root
spring.datasource.two.password=123456
spring.datasource.two.driverClassName=com.mysql.cj.jdbc.Driver
spring.datasource.two.type=com.alibaba.druid.pool.DruidDataSource
```
#### 数据源配置类  
这两个数据源配置类在config目录下
```java
package com.sogou.bootdemo3.config;

import org.apache.ibatis.annotations.One;
import org.apache.ibatis.jdbc.SQL;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.PathResource;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.stereotype.Repository;

import javax.sql.DataSource;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-11-19 17:15
 **/

@Configuration
@MapperScan(basePackages = "com.sogou.bootdemo3.dao1",sqlSessionFactoryRef = "sqlSessionFactoryOne"
        ,sqlSessionTemplateRef ="sqlSessionTemplateOne",annotationClass = Repository.class)
public class MyBatisConfigOne {

    //配置数据源1
    @Bean("datasourceOne")
    @ConfigurationProperties("spring.datasource.one")
    public DataSource getDataSourceOne(){
        return DataSourceBuilder.create().build();
    }

    //配置sqlSessionFactory
    @Bean("sqlSessionFactoryOne")
    public SqlSessionFactory getSqlSessionFactoryOne(@Qualifier("datasourceOne") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        return sqlSessionFactoryBean.getObject();

    }

    //配置sqlSessionTemplate
    @Bean("sqlSessionTemplateOne")
    public SqlSessionTemplate getSqlSessionTemplateOne(@Qualifier("sqlSessionFactoryOne") SqlSessionFactory sqlSessionFactory){
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```
这里说明一下，@Configuration注解说明这个类是一个配置类，@MapperScan注解允许通过扫描自动加载mybatis的Mapper，basePackages是Mapper所在的路径。如果项目中不存在多个SqlSessionFactory或SqlSessionTemplate，可以完全不用配置sqlSessionFactoryRef和sqlSessionTemplateRef，这里需要配置多个数据源，所以需要自定义SqlSessionFactory或SqlSessionTemplate。如果同时存在SqlSessionFactory和SqlSessionTemplate，则会后者生效。annotationClass设置了注解限制，限制只扫描@Repository  
**编写另一个数据源类**  
```java
package com.sogou.bootdemo3.config;

import jdk.jfr.Registered;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.stereotype.Repository;

import javax.sql.DataSource;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-11-19 17:29
 **/
@Configuration
@MapperScan(basePackages = "com.sogou.bootdemo3.dao2",sqlSessionFactoryRef = "sqlSessionFactoryTwo"
        ,sqlSessionTemplateRef = "sqlSessionTemplateTwo",annotationClass = Repository.class)
public class MyBatisConfigTwo {
    //配置数据源2

    @Bean("datasourceTwo")
    @ConfigurationProperties("spring.datasource.two")
    public DataSource getDataSourceTwo(){
        return DataSourceBuilder.create().build();
    }


    @Bean("sqlSessionFactoryTwo")
    public SqlSessionFactory getSqlSessionFactoryTwo(@Qualifier("datasourceTwo") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        return sqlSessionFactoryBean.getObject();
    }

    @Bean("sqlSessionTemplateTwo")
    public SqlSessionTemplate getSqlSessionTemplateTwo(@Qualifier("sqlSessionFactoryTwo") SqlSessionFactory sqlSessionFactory){
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```
### Mapper和对应xml  
因为需要配置多数据源，所以将mapper和xml文件分别写在dao1目录和dao2目录下  
```java
package com.sogou.bootdemo3.dao1;

import com.sogou.bootdemo3.pojo.Teacher;
import org.springframework.stereotype.Repository;

import java.util.List;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-11-19 17:31
 **/
@Repository("one")
public interface TeacherMapperOne {
    List<Teacher> getAllTeacher();
}
```
这里使用@Repository是为了和上述数据源配置类里面的annotationClass限制对应起来，也可以使用mybatis自带注解@Mapper。
对应的xml文件  
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sogou.bootdemo3.dao1.TeacherMapperOne">

    <select id="getAllTeacher" resultType="com.sogou.bootdemo3.pojo.Teacher">
                select * from tb_teacher;
    </select>
</mapper>
```
**编写另一个mapper和xml**  
```java
package com.sogou.bootdemo3.dao2;

import com.sogou.bootdemo3.pojo.Teacher;
import org.springframework.stereotype.Repository;

import java.util.List;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-11-19 17:32
 **/
@Repository("two")
public interface TeacherMapperTwo {
    List<Teacher> getAllTeacher();
}
```
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sogou.bootdemo3.dao2.TeacherMapperTwo">

    <select id="getAllTeacher" resultType="com.sogou.bootdemo3.pojo.Teacher">
                select * from tb_teacher;
    </select>
</mapper>
```
### 接口实现类  
这两个接口实现类是对上述mapper的具体实现，写在impl目录下。这样做是为了利用多态  
```java
package com.sogou.bootdemo3.impl;

import com.sogou.bootdemo3.dao1.TeacherMapperOne;
import com.sogou.bootdemo3.pojo.Teacher;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-11-19 17:36
 **/
@Service("teacherOne")
public class TeacherMapperOneImpl implements TeacherMapperOne {

    @Autowired
    @Qualifier("one")
    private TeacherMapperOne teacherMapperOne;

    @Override
    public List<Teacher> getAllTeacher() {
        return teacherMapperOne.getAllTeacher();
    }
}
```
```java
package com.sogou.bootdemo3.impl;

import com.sogou.bootdemo3.dao2.TeacherMapperTwo;
import com.sogou.bootdemo3.pojo.Teacher;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

import java.util.List;


/**
 * author liujinxi@sogou-inc.com
 * date 2020-11-19 17:38
 **/
@Service("teacherTwo")
public class TeacherMapperTwoImpl implements TeacherMapperTwo {


    @Autowired
    @Qualifier("two")
    private TeacherMapperTwo teacherMapperTwo;

    @Override
    public List<Teacher> getAllTeacher() {
        return teacherMapperTwo.getAllTeacher();
    }
}
```
### 对应的javaBean  
对应的实体类写在pojo目录下  
```java
package com.sogou.bootdemo3.pojo;

import org.springframework.util.StringUtils;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-11-19 17:34
 **/
public class Teacher {

    private int id;
    private String name;
    private String sex;

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

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    @Override
    public String toString() {
        return "Teacher{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", sex='" + sex + '\'' +
                '}';
    }
}
```
### controller层  
这里使用controller作为测试，编写controller  
```java
package com.sogou.bootdemo3.controller;

import com.sogou.bootdemo3.dao1.TeacherMapperOne;
import com.sogou.bootdemo3.dao2.TeacherMapperTwo;
import com.sogou.bootdemo3.pojo.Teacher;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-11-19 17:40
 **/


@RestController
public class HelloController {


    @Autowired
    @Qualifier("teacherOne")
    private TeacherMapperOne teacherMapperOneImpl;
    @Autowired
    @Qualifier("teacherTwo")
    private TeacherMapperTwo teacherMapperTwoImpl;


    @GetMapping("/one")
    public Object getAllTeacherOne(){

        List<Teacher> allTeahcer = teacherMapperOneImpl.getAllTeacher();
        return allTeahcer;
    }


    @GetMapping("/two")
    public Object getAllTeacherTwo(){
        List<Teacher> allTeacher = teacherMapperTwoImpl.getAllTeacher();
        return allTeacher;
    }

}
```
通过访问localhost:8080/one和localhost:8080/two可以发现mybatis多数据源配置成功  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/11/20/1605843392484-1605843392486.png)
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/11/20/1605843376494-1605843376496.png)  
**其实关于多数据源，复杂的应该直接使用分布式数据库中间件，简单的在考虑多数据源**