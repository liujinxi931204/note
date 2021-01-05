## 简介  
Mybatis-Plus，简称MP，是一个Mybatis的增强工具，在Mybatis的基础上只做增强不做改变，为简化开发、提高效率而生  
### 支持数据库  
任何能使用mybatis进行crud，并且支持标准sql的数据库  
### 框架结构  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2021/01/05/1609810091271-1609810091331.png)  
### 安装  
```xml
<!--        引入mybatis-plus的starter-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.4.1</version>
        </dependency>
```
### 数据准备  
```sql
drop table if exists user;
CREATE TABLE `user` (
  `id` int(11) NOT NULL,
  `name` varchar(30) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `email` varchar(30) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8


drop table if exists book;
CREATE TABLE `book` (
  `book_id` int(10) NOT NULL AUTO_INCREMENT,
  `book_name` varchar(400) DEFAULT NULL,
  `price` float DEFAULT NULL,
  `content` text,
  PRIMARY KEY (`book_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```  
### Mapper CRUD接口 
**说明**  
通用CURD封装BaseMapper接口，为Mybatis-Plus启动时自动解析实体表关系映射转化为Mybatis内部对象注入容器  
BaseMapper接口的全限定名为`com.baomidou.mybatisplus.core.mapper.BaseMapper<T>`，该接口提供了插入、修改、删除和查询接口  
如下  
#### select 查询  
```java
// 根据 ID 查询
T selectById(Serializable id);
// 根据 entity 条件，查询一条记录
T selectOne(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
 
// 查询（根据ID 批量查询）
List<T> selectBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
// 根据 entity 条件，查询全部记录
List<T> selectList(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 查询（根据 columnMap 条件）
List<T> selectByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
// 根据 Wrapper 条件，查询全部记录
List<Map<String, Object>> selectMaps(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询全部记录。注意： 只返回第一个字段的值
List<Object> selectObjs(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
 
// 根据 entity 条件，查询全部记录（并翻页）
IPage<T> selectPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询全部记录（并翻页）
IPage<Map<String, Object>> selectMapsPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
 
// 根据 Wrapper 条件，查询总记录数
Integer selectCount(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```  
#### insert插入数据  
```java
// 插入一条记录，entity 为实体对象
int insert(T entity);
```  
#### update更新数据  
```java
// 根据 whereEntity 条件，更新记录
int update(@Param(Constants.ENTITY) T entity, @Param(Constants.WRAPPER) Wrapper<T> updateWrapper);
// 根据 ID 修改
int updateById(@Param(Constants.ENTITY) T entity);
```  
#### delete删除数据  
```java
// 根据 entity 条件，删除记录
int delete(@Param(Constants.WRAPPER) Wrapper<T> wrapper);
// 删除（根据ID 批量删除）
int deleteBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
// 根据 ID 删除
int deleteById(Serializable id);
// 根据 columnMap 条件，删除记录
int deleteByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
```  
使用Mapper CRUD接口首先需要自定义mapper接口继承BaseMapper  
```java
public interface UserMapper extends BaseMapper<User> {

}
```  
然后在MybatisDemo2Application上使用@MapperScan扫面自定义的mapper接口  
```java
@SpringBootApplication
@MapperScan("com.sogou.mybatisdemo2.mapper")
public class MybatisDemo2Application {

    public static void main(String[] args) {
        ConfigurableApplicationContext run = SpringApplication.run(MybatisDemo2Application.class, args);
    }

}
```  
可以看到上面的代码中没有任何自定义的方法，所有的方法均从BaseMapper中继承而来  
#### select 简单查询  
在查询中select语句主要作用是查询，Wrapper对象的作用地是构建查询条件

















