## 缓存概念  
### 什么是缓存
经过分类，可以将缓存分为：  
+ **硬件缓存**：一般指机器上的CPU、硬盘等等组件的缓存区间，一般是利用的内存作为一块中转区域，都通过内存交互信息，减少系统负载，提供传输效率  
+ **客户端缓存**：一般指的是某些应用，例如浏览器等等，都是在加载一次数据后将数据临时存储到本地，当再次访问时候先检查本地缓存中是否存在，存在就不去远程重新拉取，而是直接读取缓存数据，这样来减少远端服务器压力和加快载入速度  
+ **服务端缓存**：一般指在远端服务器上，考虑到客户端请求量多，某些数据请求量大，这些热点数据经常要到数据库中读取数据，给数据库造成压力，还有就是IO、网络等原因造成一定延迟，响应客户端较慢。所以，在一些不考虑实时性的数据中，经常将这些数据存在内存中(内存速度非常快)，当请求时，能够直接读取内存中的数据及时响应  
### 为什么使用缓存  
用缓存，主要解决高性能、高并发与减少数据库压力。缓存本质就是将数据存储在内存中，当数据没有发生本质变化的时候，应该尽量避免直接连接数据库进行查询，因为并发高时很可能会将数据库压塌，而是应去读取数据，只有缓存中未查找到时再去数据库中查询，这样就大大降低了数据库的读写次数，增加系统的性能和能提供的并发量  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/11/30/1606724988682-1606724988733.png)  
### 缓存的优缺点  
#### 优点  
+ 加快响应速度  
+ 减少了对数据库的读操作，数据库的压力降低  
#### 缺点  
+ 内存容量相对于硬盘小  
+ 缓存中的数据可能与数据库中数据不一致  
+ 因为内存断电就清空数据，存放到内存中的数据可能丢失  
## 缓存后可能遇见的问题  
### 缓存穿透  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/11/30/1606725860708-1606725860711.png)  
**缓存穿透**：指查询一个一定不存在的数据，由于缓存是不命中时需要从数据库查询，查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要去到数据库去查询，造成缓存穿透  
#### 缓存穿透几种解决办法  
+ 缓存空值，在从DB查询对象为空时，也要将控制存入缓存，具体的值需要使用特殊的标识，能和真正缓存的数据区分开，另外也要将其过期时间设为较短  
+ 使用布隆过滤器，布隆过滤器能判断一个key一定不存在(不保证一定存在，因为布隆过滤器结构原因，不能删除，但是旧值可能被新值替换，而将旧值删除后它可能依旧判断其可能存在)，在缓存的基础上，构建布隆过滤器数据结构，在布隆过滤器中存储对应的key，如果存在，则说明key对应的值为空  
### 缓存击穿  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/11/30/1606726327408-1606726327410.png)  
**缓存击穿**：某个key非常热点，访问非常频繁，处于集中式高并发访问的情况，当这个key在失效的瞬间，大量的请求就击穿了缓存，直接请求数据库，就像是在一道屏障上凿开了一个洞  
#### 缓存击穿几种解决办法  
+ 设置二级缓存，或者设置热点缓存永不过期，需要根据实际情况进行配置  
+ 使用互斥锁，在执行过程中，如果缓存过期，那么先获取分布式锁，在执行从数据库中加载数据，如果找到数据就存入缓存，没有就继续该有的动作，在这个过程中能保证只有一个线程操作，避免了对数据库的大量请求  
### 缓存雪崩  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/11/30/1606726885946-1606726885947.png)  
**缓存雪崩**：当缓存服务器重启、或者大量缓存集中在某一个时段失效，这样失效的时候，也会给后端系统带来很大的压力，造成数据库后端故障，从而引起应用服务器雪崩  
#### 缓存雪崩几种解决办法  
+ 缓存组件设计高可用，缓存高可用是指，存储缓存的组件高可用，能够防止单点故障、机器故障、机房宕机等一系列问题。例如redis 的sentinel和redis cluster  
+ 请求限流与服务熔断降级机制，限制服务请求次数，当服务不可用时快速熔断降级  
+ 设置缓存过期时间一定的随机分布，避免集中在同一时间缓存失效  
+ 定时更新策略，对于实时性要求不高的数据，定时进行更新  
### 缓存一致性  
使用缓存很大可能导致数据不一致问题：  
+ 更新数据库成功->更新缓存失败->数据不一致  
+ 更新缓存成功->更新数据库失败->数据不一致  
+ 更新数据库成功->淘汰缓存失败->数据不一致  
+ 淘汰缓存成功->更新数据库失败->数据查询mis  
## Spring的缓存抽象  
Spring框架自身并没有实现缓存解决方案，但是从3.1开始定义了org.springframework.cache.Cache和org.srpingframeswork.cache.CacheManager接口，提供了对缓存功能的声明，能够与多种流行的缓存实现集成  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/11/30/1606727529808-1606727529809.png)  
### 几个重要概念&缓存注解  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/11/30/1606727579909-1606727579910.png)  
#### Cacheable注解属性简介  
每次调用目标方法之前都会根据给定的方法参数检查是否方法已经进行了缓存  
cacheNames/value:指定缓存组件的名字，必须至少指定一个  
```java
@Cacheable(value="cache1")
@Cacheable(value={"cache1,cache2"})
```
key:缓存数据时使用的key，可以根据该属性进行自定义设置，默认使用方法的参数  
```java
@Cacheable(value=”testcache”,key=”#userName”)
@Cacheable(value=”testcache”,key=”#root.args[0]”)
@Cacheable(value=”testcache”,key=”#root.methodName+'['+#id+']'”)
```
keyGenerator:key的生成器，可以自己指定keyGenerator组件id(自定义keyGenerator，不能同时和key使用)  
cacheManager：指定缓存管理器；cacheResolver指定获取缓存解析器，二者二选一  
condition：缓存的条件，可以为空，也可以使用SpEL表达式编写，返回true或者false，只有为true才进行缓存，在调用方法之前、之后都能判断  
```java
@Cacheable(value=”testcache”,condition=”#id>2”)
@Cacheable(value=”testcache”,condition=”#a0>2”)
```
unless:当unless的条件为true时，方法的返回值就不缓存。该表达式只在方法执行之后判断，此时可以拿到返回值result进行判断。条件为true时不会缓存，false才会缓存  
```java
@Cacheable(value=”testcache”,unless=”#result == null”)
```
sync：是否启用异步模式。默认采用同步方式，在方法执行完将结果放入缓存，可以设置为true，启用异步模式。需要注意的是，异步模式不能与unless同时使用。
#### Cache SpEL
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/11/30/1606728774849-1606728774851.png)  
#### @CachePut注解使用  
既调用方法，又更新缓存(先调用方法，然后将方法的返回结果放进缓存)  
注意 **@CachePut中的key是不能使用#{result}的**  
#### @CacheEvict注解使用  
默认先执行方法，然后根据参数作为key从缓存中删除数据。如果方法执行过程中抛出了异常，则不会删除缓存中目标数据  
allEntries:默认false，表示是否全部删除对应缓存组件中的数据  
beforeInvocation：默认false，表示是否在方法执行执行删除缓存  
#### @Cacheing注解  
该注解用来构建复杂规则的缓存用例  
```java
@Caching(
            cacheable ={
                    @Cacheable(value ="emp",key = "#lastName")
            },
            put = {
                    @CachePut(value = "emp",key = "#result.id"),
                    @CachePut(value = "emp",key = "#result.email")
            }
    )
public Employee getEmpByLastName(String lastName){
     System.out.println("调用复杂缓存方法");
     return employeeMapper.getEmpByLastName(lastName);
 }
```
第一次按照lastName进行缓存的同时(cacheable 注解)，@CachePut注解也起作用–分别以id和email为key在缓存中放入数据。再次进行查询的时候方法仍然会调用，因为@CachePut注解一直起作用！  
#### @CacheConfig注解使用  
同一个类中不同方法汇总缓存组件名字一般相同，可以使用@CacheConfig注解作用在类上配置共同属性值，默认对该类的所有方法起作用  

## 整合第三方缓存redis  
### 目录结构  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/11/30/1606729744873-1606729744874.png)
### maven引入相关依赖  
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
    <artifactId>redisdemo2</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>redisdemo2</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>11</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
<!--        使用默认Lettuce时，若配置spring.redis.lettuce.pool必须配置该依赖-->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.4</version>
        </dependency>
<!--        引入druid数据库连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
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
### 配置数据库、redis、mybatis、cache相关配置  
```properties
#数据库配置
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.url=jdbc:mysql://10.160.58.128:3306/test_ljx
spring.datasource.username=root
spring.datasource.password=123456

#redis配置
spring.redis.host=10.160.58.128
spring.redis.port=6379
spring.redis.lettuce.pool.max-active=8
spring.redis.lettuce.pool.max-idle=8
spring.redis.lettuce.pool.max-wait=-1
spring.redis.lettuce.pool.min-idle=0

#mybatis配置,如果不指定这个配置，pojo类上的Alias会找不到Bean
mybatis.type-aliases-package=com.sogou.redisdemo2.pojo

#cache配置
spring.cache.redis.use-key-prefix=true
spring.cache.redis.time-to-live=1d
```
### 自定义配置类  
```java
package com.sogou.redisdemo2.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.jsontype.impl.LaissezFaireSubTypeValidator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.util.Arrays;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-11-29 16:35
 **/
@Configuration
public class redisConfig {

//    由于原生redis自动装配，在存储key和value的时候，没有设置序列化，创建自己的redisTemplate
    @Bean
    public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory redisConnectionFactory){
        RedisTemplate<String,Object> redisTemplate=new RedisTemplate<>();
//配置连接工厂
        redisTemplate.setConnectionFactory(redisConnectionFactory);
//        使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值，如果不适用的话，默认使用JDK序列化
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer=new Jackson2JsonRedisSerializer<>(Object.class);
//        ObjectMapper类是Jackson库的主要类。它提供一些功能将转换成Java对象匹配JSON结构
        ObjectMapper objectMapper = new ObjectMapper();
//        指定要序列化的域，filed、get和set，以及修饰符范围，ANY是都包括private、public
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
//        指定序列化输入的类型，类必须是非final修饰的类，比如String、Integer等会跑出错误        objectMapper.activateDefaultTyping(LaissezFaireSubTypeValidator.instance,ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
//        设置key使用StringRedisSerializer序列化
        redisTemplate.setKeySerializer(new StringRedisSerializer());
//        设置value使用Jackson2JsonRedisSerializer序列化
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
//        设置hash key使用StringRedisSerializer序列化
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
//        设置hash value使用jackson2JsonRedisSerializer序列化
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }


    /**
     *
     * 申明缓存管理器，会创建一个切面（aspect）并触发Spring缓存注解的切点（pointcut）
     * 根据类或者方法所使用的注解以及缓存的状态，这个切面会从缓存中获取数据，将数据添加到缓存之中或者从缓存中移除某个值
     * @param redisConnectionFactory
     * @return
     */
    @Bean
    public RedisCacheManager redisCacheManager(RedisConnectionFactory redisConnectionFactory){
        return RedisCacheManager.create(redisConnectionFactory);
    }

//    自定义缓存的key生成器
//    @Bean("myKey")
//    public KeyGenerator keyGenerator(){
//        return (o, method, objects) -> method.getName()+"["+ Arrays.asList(objects).toString()+"]";
//    }

}
```
### pojo类  
package com.sogou.redisdemo2.pojo;

import org.apache.ibatis.type.Alias;

import java.io.Serializable;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-11-27 17:53
 **/

@Alias("teacher")
public class Teacher implements Serializable {

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

### mybatis配置的dao层和xml映射文件  
```java
package com.sogou.redisdemo2.dao;

import com.sogou.redisdemo2.pojo.Teacher;
import org.springframework.stereotype.Repository;

import java.util.List;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-11-27 19:17
 **/
@Repository("teacherDao")
public interface TeacherDao {

    public List<Teacher> getAllTeacher();
    public int updateTeacher(Teacher teacher);
    public Teacher getTeacherById(int id);

}

```
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sogou.redisdemo2.dao.TeacherDao">

    <select id="getAllTeacher" resultType="teacher">
                select * from tb_teacher;
    </select>

    <select id="getTeacherById" parameterType="int">
        select * from tb_teacher where id=#{id}
    </select>

    <update id="updateTeacher" parameterType="Teacher">
        update tb_teacher set name=#{name} where id=#{id}
    </update>
</mapper>
```
### service层  
```java
package com.sogou.redisdemo2.impl;

import com.sogou.redisdemo2.dao.TeacherDao;
import com.sogou.redisdemo2.pojo.Teacher;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.AutoConfigureAfter;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.cache.annotation.Caching;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-11-27 19:18
 **/
@Service("teacherDaoImpl")
public class TeacherDaoImpl  implements TeacherDao {

    @Autowired
    @Qualifier("teacherDao")
    private TeacherDao teacherDao;

    @Override
    public List<Teacher> getAllTeacher() {
        return teacherDao.getAllTeacher();
    }

    @Override
    public int updateTeacher(Teacher teacher) {
        return teacherDao.updateTeacher(teacher);
    }

    @Override
    public Teacher getTeacherById(int id) {
        return teacherDao.getTeacherById(id);
    }
    
}

```
### controller层  
```java
package com.sogou.redisdemo2.controller;

import com.sogou.redisdemo2.dao.TeacherDao;
import com.sogou.redisdemo2.pojo.Teacher;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.cache.annotation.Caching;
import org.springframework.core.style.ToStringCreator;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;


import java.util.List;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-11-27 19:24
 **/

@RestController
public class helloController {

    @Autowired
    @Qualifier("teacherDaoImpl")
    private TeacherDao teacherDao;


    @GetMapping("/get")
    @Cacheable(value = "teacherCache",key = "#root.target")
    public Object getAllTeacher(){
        System.out.println("调用了方法，没有使用缓存");
        List<Teacher> allTeacher = teacherDao.getAllTeacher();
        return allTeacher;
    }

    @GetMapping("/update/{name}/{id}")
    @Caching(
//            每次更新之后，更新#id缓存
            put = {
                    @CachePut(value = "teacherCache",key = "#id")
            },
//            每次更新之后，清除#root.target缓存，为的是查询所有的时候直接从数据库获取到最新的
            evict = {
                    @CacheEvict(value = "teacherCache",key = "#root.target")
            }
    )
    public Teacher updateTeacher(@PathVariable("name")String name,@PathVariable("id")int id){
        System.out.println("调用了方法，没有使用缓存");
        Teacher teacher = teacherDao.getTeacherById(id);
        teacher.setId(id);
        teacher.setName(name);
        teacherDao.updateTeacher(teacher);
        return teacher;
    }


    @GetMapping("/get/{id}")
    @Cacheable(value = "teacherCache",key = "#id")
    public Teacher getTeacherById(@PathVariable("id") int id){
        System.out.println("调用了方法，没有使用缓存");
        return teacherDao.getTeacherById(id);
    }

    @Override
//    重写toString方法，为的是使用缓存注解的时候key可以使用#root
//    如果不重写toString方法，会存在Cannot convert cache key的错误
    public String toString() {
        return "helloController";
    }
}

```