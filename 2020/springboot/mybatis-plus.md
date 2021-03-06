## 简介  
Mybatis-Plus，简称MP，是一个Mybatis的增强工具，在Mybatis的基础上只做增强不做改变，为简化开发、提高效率而生  
### 支持数据库  
任何能使用mybatis进行crud，并且支持标准sql的数据库  
### 框架结构  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2021/01/05/1609810091271-1609810091331.png)  
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
+ selectById:根据ID查询  
```java
@SpringBootTest
class Select0Test {
 
    @Autowired
    private UserMapper userMapper;
 
    @Test
    void contextLoads() {
        User userBean = userMapper.selectById(1);
        System.out.println(userBean);
    }
}
```
+ selectBatchIds:根据ID批量查询，即一次传入多个ID  
```java
@SpringBootTest
class Select1Test {
 
    @Autowired
    private UserMapper userMapper;
 
    @Test
    void contextLoads() {
        List<Integer> ids = Arrays.asList(1, 2, 3);
        List<User> userBeanList = userMapper.selectBatchIds(ids);
        userBeanList.forEach(System.out::println);
    }

}
```
+ selectOne:根据构建的Wrapper条件查询数据，且只返回一个结果对象  
```java
@SpringBootTest
class Select8Test {
 
    @Autowired
    private UserMapper userMapper;
 
    @Test
    void contextLoads() {
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        //相当于select * from user where id<=1;
        //wrapper的条件构造函数
        wrapper.le("id", 1);
        User userBean = simpleMapper.selectOne(wrapper);
        System.out.println(userBean);
    }
 
}
```
+ selectCount:根据构建的Wrapper条件对象查询数据条数  
```java
@SpringBootTest
class Select8Test {
 
    @Autowired
    private UserMapper userMapper;
 
    @Test
    void contextLoads() {
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        //相当于select count(1) from user where age>=20;
        //wrapper的条件构造函数
        wrapper.ge("age", 20);
        int count = simpleMapper.selectCount(wrapper);
        System.out.println(count);
    }
}
```
#### select复杂查询  
+ selectList:根据entity条件，查询全部记录  
```java
@SpringBootTest
class Select8Test {
 
    @Autowired
    private UserMapper userMapper;
 
    @Test
    void contextLoads() {
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        //相当于select * from user where age=20;
        //wrapper的条件构造函数
        wrapper.eq("age", 20);
        List<User> userList=userMapper.selectList(wrapper);
        userList.forEach(System.out::println);
    }
}
```
+ selectByMap:根据查询条件(columnMap条件)，查询全部记录  
```java
@SpringBootTest
class Select8Test {
 
    @Autowired
    private UserMapper userMapper;
 
    @Test
    void contextLoads() {
        Hash<String,Object> map=new HashMap<>();
        map.put("id",1);
        map.put("age",18)
        //相当于select * from user where id=1 and age=18;
        List<User> userList=userMapper.selectByMap(map);
        userList.forEach(System.out::println);
    }
}
```
#### 动态select查询  
```java
@SpringBootTest
class Select2Test {
 
    @Autowired
    private SimpleMapper simpleMapper;
 
    @Test
    void contextLoads() {
        query(1, null, null);
        query(1, "赫仑", null);
        query(1, "赫仑", 27);
    }
 
    private void query(int userId, String name, Integer age) {
        System.out.println("\n查询数据：");
        QueryWrapper<UserBean> wrapper = new QueryWrapper<>();
        wrapper.eq("user_id", userId);
 
        // 第一个参数为是否执行条件，为true则执行该条件
        // 下面实现根据 name 和 age 动态查询
        wrapper.eq(StringUtils.isNotEmpty(name), "name", name);
        wrapper.eq(null != age, "age", age);
 
        List<UserBean> userBeanList = simpleMapper.selectList(wrapper);
        for(UserBean userBean : userBeanList) {
            System.out.println(userBean);
        }
    }
 
}
```
上面的查询中，会发现"wrapper(null!=age,"age",age);"语句的第一个参数是一个布尔值，当该布尔值为true时，则该条件应用到结果集中，类似于Mybatis xml文件中的<if>标签，等同于如下  
```xml
<if test="null!=age">
    and age=#{age}
</if>
```
#### select 分页查询  
在进行分页查询的时候，需要配置分页插件，因此需要通过@Configuration和@Bean注解来添加配置  
```java
@Configuration
public class MybatisPlusConfig {
 
    /**
     * 分页插件。如果你不配置，分页插件将不生效
     */
    @Bean
    public MybatisPlusInterceptor paginationInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        // 指定数据库方言为 MYSQL
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```
+ selectPage:根据entity条件，查询全部记录  
```java
@SpringBootTest
class Select8Test {
 
    @Autowired
    private UserMapper userMapper;
 
    @Test
    void contextLoads() {
        System.out.println("------------selectAll method test--------------");
        QueryWrapper<User> userQueryWrapper = new QueryWrapper<>();
        userQueryWrapper.isNotNull("id");
//        创建分页对象(1表示第一页，2表示每页大小为2)
        Page<User> userPage = new Page<>(1,2);
        Page<User> userResult = userMapper.selectPage(userPage, userQueryWrapper);
        List<User> records = userResult.getRecords();
        records.forEach(System.out::println);
    }
}
```
+ selectMapsPage:根据wrapper条件，查询全部记录  
这个方法的使用和上面一致，仅仅是返回类型不同  
```java
@SpringBootTest
class Select8Test {
 
    @Autowired
    private UserMapper userMapper;
 
    @Test
    void contextLoads() {
        System.out.println("------------selectAll method test--------------");
        QueryWrapper<User> userQueryWrapper = new QueryWrapper<>();
        userQueryWrapper.isNotNull("id");
//        创建分页对象(1表示第一页，2表示每页大小为2)
        Page<Map<String,Object>> page=new Page<>(2,2);
        Page<Map<String, Object>> userResult = userMapper.selectMapsPage(page,userQueryWrapper);
        List<Map<String, Object>> records = userResult.getRecords();
        records.forEach(System.out::println);
        }
    }
}
```
#### insert保存数据  
+ insert：插入一条记录  
需要注意的是，insert一个实体对象需要使用@TableName和@TableId注解，说明该实体对应哪个表，并且哪个字段是ID字段  
```java
@SpringBootTest
class Select8Test {
 
    @Autowired
    private UserMapper userMapper;
 
    @Test
    void contextLoads() {
        User user = new User();
        user.setId(6);
        user.setName("Henry");
        user.setAge(25);
        user.setEmail("test1@baomidou.com");
        int insert = userMapper.insert(user);
        }
    }
}
```
#### update更新数据  
+ update：根据Entity条件更新记录  
```java
@SpringBootTest
class Select8Test {
 
    @Autowired
    private UserMapper userMapper;
 
    @Test
    void contextLoads() {
        User user = new User();
        user.setAge(22);
        QueryWrapper<User> userQueryWrapper = new QueryWrapper<>();
        userQueryWrapper.eq("id",3);
//相当于update user set age=22 where id=3；
//就是根据条件更新某一条记录卡
        userMapper.update(user, userQueryWrapper);
        }
    }
}
```
+ updateById:根据id更新记录  
**注意这里传入的参数是实体类而不是具体的id数值**  
```java
@SpringBootTest
class Select8Test {
 
    @Autowired
    private UserMapper userMapper;
 
    @Test
    void contextLoads() {
        User user = new User();
        user.setId(7);
        user.setName("Jerry");
        user.setAge(25);
        user.setEmail("test1@baomidou.com");
//相当于update user set name=xxx and age=xxx and email=xxx where id=3；
//就是根据条件更新某一条记录卡
        userMapper.update(user, userQueryWrapper);
        }
    }
}
```
#### delete删除数据  
+ delete:根据wrapper条件删除记录  
```java
@SpringBootTest
class Select8Test {
 
    @Autowired
    private UserMapper userMapper;
 
    @Test
    void contextLoads() {
        QueryWrapper<User> userQueryWrapper = new QueryWrapper<>();
        userQueryWrapper.eq("id",3);
//相当于delete from user where id=3；
//就是根据条件更新某一条记录卡
        userMapper.delete(userQueryWrapper);
        }
    }
}
```
+ deleteBatchIds:根据id批量删除  
```java
@SpringBootTest
class Select8Test {
 
    @Autowired
    private UserMapper userMapper;
 
    @Test
    void contextLoads() {
        List<Integer> idList=Arrays.asList(1,2,3,4);
        userMapper.deleteBatchIds(idList);
        }
    }
}
```
+ deleteById:根据id删除  
```java
@SpringBootTest
class Select8Test {
 
    @Autowired
    private UserMapper userMapper;
 
    @Test
    void contextLoads() {
        userMapper.deleteById(999);
        }
    }
}
```
+ deleteByMap:根据columnMap删除  
```java
@SpringBootTest
class Select8Test {
 
    @Autowired
    private UserMapper userMapper;
 
    @Test
    void contextLoads() {
        Map<String,Object> userMap=new HashMap<>();
        userMap.put("age",20);
//相当于执行delete from user where age=20;
        userMapper.deleteByMap(userMap);
        }
    }
}
```
### Service CRUD接口  
MyBatis Plus提供了除通用的Mapper接口(即BaseMapper)，该接口对应的DAO层。除了BaseMapper接口之外，还提供了IService接口，该接口对应Service层。MyBatis Plus通用的Service CRUD实现了IService接口，进一步封装了CRUD。为了避免与BaseMapper中定义的 方法混淆，该接口使用get(查询单行)、remove(删除)、list(查询集合)和page(分页)前缀命名的方式进行区别  
MyBatis-Plus使用ServiceImpl类实现了IService接口，因此在使用时需要service层的类继承ServiceImpl类  
部分源码如下  
```java
//IService实现类(泛型：M是Mapper对象，T是实体)
public class ServiceImpl<M extends BaseMapper<T>, T> implements IService<T> {
    protected Log log = LogFactory.getLog(this.getClass());
    @Autowired
    protected M baseMapper;
    protected Class<T> entityClass = this.currentModelClass();
    protected Class<T> mapperClass = this.currentMapperClass();

    public ServiceImpl() {
    }  
    //忽略其他代码
```
从源码上看，IService内部还是基于BaseMapper进行封装的  
IService提供以下方法：  
**保存数据(save)**  
```java
// 插入一条记录（选择字段，策略插入）
boolean save(T entity);
// 插入（批量）
boolean saveBatch(Collection<T> entityList);
// 插入（批量）
boolean saveBatch(Collection<T> entityList, int batchSize);
```
**保存或更新(saveOrUpdate)**  
```java
// TableId 注解存在更新记录，否插入一条记录
boolean saveOrUpdate(T entity);
// 根据updateWrapper尝试更新，否继续执行saveOrUpdate(T)方法
boolean saveOrUpdate(T entity, Wrapper<T> updateWrapper);
// 批量修改插入
boolean saveOrUpdateBatch(Collection<T> entityList);
// 批量修改插入
boolean saveOrUpdateBatch(Collection<T> entityList, int batchSize);
```
**移除数据(remove)**  
```java
// 根据 entity 条件，删除记录
boolean remove(Wrapper<T> queryWrapper);
// 根据 ID 删除
boolean removeById(Serializable id);
// 根据 columnMap 条件，删除记录
boolean removeByMap(Map<String, Object> columnMap);
// 删除（根据ID 批量删除）
boolean removeByIds(Collection<? extends Serializable> idList);
```
**更新数据(update)**  
```java
// 根据 UpdateWrapper 条件，更新记录 需要设置sqlset
boolean update(Wrapper<T> updateWrapper);
// 根据 whereEntity 条件，更新记录
boolean update(T entity, Wrapper<T> updateWrapper);
// 根据 ID 选择修改
boolean updateById(T entity);
// 根据ID 批量更新
boolean updateBatchById(Collection<T> entityList);
// 根据ID 批量更新
boolean updateBatchById(Collection<T> entityList, int batchSize);
```
**获取单条数据(get)**  
```java
// 根据 ID 查询
T getById(Serializable id);
// 根据 Wrapper，查询一条记录。结果集，如果是多个会抛出异常，随机取一条加上限制条件 wrapper.last("LIMIT 1")
T getOne(Wrapper<T> queryWrapper);
// 根据 Wrapper，查询一条记录
T getOne(Wrapper<T> queryWrapper, boolean throwEx);
// 根据 Wrapper，查询一条记录
Map<String, Object> getMap(Wrapper<T> queryWrapper);
// 根据 Wrapper，查询一条记录
<V> V getObj(Wrapper<T> queryWrapper, Function<? super Object, V> mapper);
```
**获取数据列表(list)**  
```java
// 查询所有
List<T> list();
// 查询列表
List<T> list(Wrapper<T> queryWrapper);
// 查询（根据ID 批量查询）
Collection<T> listByIds(Collection<? extends Serializable> idList);
// 查询（根据 columnMap 条件）
Collection<T> listByMap(Map<String, Object> columnMap);
// 查询所有列表
List<Map<String, Object>> listMaps();
// 查询列表
List<Map<String, Object>> listMaps(Wrapper<T> queryWrapper);
// 查询全部记录
List<Object> listObjs();
// 查询全部记录
<V> List<V> listObjs(Function<? super Object, V> mapper);
// 根据 Wrapper 条件，查询全部记录
List<Object> listObjs(Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询全部记录
<V> List<V> listObjs(Wrapper<T> queryWrapper, Function<? super Object, V> mapper);
```
**分页(page)**  
```java
// 无条件分页查询
IPage<T> page(IPage<T> page);
// 条件分页查询
IPage<T> page(IPage<T> page, Wrapper<T> queryWrapper);
// 无条件分页查询
IPage<Map<String, Object>> pageMaps(IPage<T> page);
// 条件分页查询
IPage<Map<String, Object>> pageMaps(IPage<T> page, Wrapper<T> queryWrapper);
```
**查询记录数(count)**  
```java
// 查询总记录数
int count();
// 根据 Wrapper 条件，查询总记录数
int count(Wrapper<T> queryWrapper);
```
**链式查询(chain query)**
```java
// 链式查询 普通
QueryChainWrapper<T> query();
// 链式查询 lambda 式。注意：不支持 Kotlin
LambdaQueryChainWrapper<T> lambdaQuery(); 
 
// 示例：
query().eq("column", value).one();
lambdaQuery().eq(Entity::getId, value).list();
```
**链式更新(chain update)**  
```java
// 链式更改 普通
UpdateChainWrapper<T> update();
// 链式更改 lambda 式。注意：不支持 Kotlin 
LambdaUpdateChainWrapper<T> lambdaUpdate();
 
// 示例：
update().eq("column", value).remove();
lambdaUpdate().eq(Entity::getId, value).update(entity);
```
#### 链式查询  
在IService中提供了一个query方法，该方法返回QueryChainWrapper对象，可以使用该对象实现链式查询，避免每次都创建QueryWrapper对象  
例如查询book表中，book_id>=2,price>=2的书籍  
```java
@SpringBootTest
class Select8Test {
 
    @Autowired
    private UserMapper userMapper;
 
    @Test
    void contextLoads() {
        List<Book> bookList = bookService.query().ge("book_id", 2).ge("price", 70).list();
        bookList.forEach(System.out::println);
        }
    }
}
```



















、