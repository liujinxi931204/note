在某些场景下可能需要配置多个数据源，使用多个数据源(例如实现数据库读写分离)来缓解系统的压力等。这里主要是springboot+mybatis实现多数据源的配置  
### 系统目录  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/11/20/1605839685773-1605839685866.png)  
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
首先编写application.properties配置文件
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