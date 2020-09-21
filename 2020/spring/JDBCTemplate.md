## 什么是JDBCTemplate  
+ Spring框架对JDBC进行封装，使用JDBCTemplate方便实现对数据库的操作  
+ 准备工作  
1. 准备jar包，引入依赖  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/09/20/1600611409859-1600611409920.png)  
2. 在Spring配置文件中配置数据库连接池（也可以使用配置jdbc.properties，然后读取到Spring的配置文件中）  
```java
<!-- 数据库连接池 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
          destroy-method="close">
        <property name="url" value="jdbc:mysql://10.160.58.128/user_db" />
        <property name="username" value="root" />
        <property name="password" value="123465" />
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
    </bean>
```  
3. 配置JDBCTemplate对象，注入dataSource对象  
```java
<!--    jdbcTemplate对象-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
<!--        注入dataSource-->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
```  
4. 创建service类，dao类，在dao类注入jdbcTemplate对象  
```java
配置文件开启注解扫描  
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">


<!--    开启注解扫描-->
    <context:component-scan base-package="com.sogou.spring"></context:component-scan>

    <!-- 数据库连接池 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
          destroy-method="close">
        <property name="url" value="jdbc:mysql://10.160.58.128/user_db" />
        <property name="username" value="root" />
        <property name="password" value="123465" />
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
    </bean>

<!--    jdbcTemplate对象-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
<!--        注入dataSource-->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
</beans>

bookService.java
@Service("bookService")
public class bookService {
    //service注入bookDao

    @Autowired
    private bookDao bookDao;
}

bookDao.java
public interface bookDao {
}

bookDaoImpl.java
@Repository("bookDaoImpl")
public class bookDaoImpl implements bookDao {

    //注入jdbcTemplate，实现数据库操作
    //使用jdbcTemplate对数据库增删改查

    @Autowired
    private JdbcTemplate jdbcTemplate;
}

```  
### JdbcTeamplate 操作数据库(添加)  
1. 对应数据库表创建出实体类  
```java
public class User {

 private int userId;
 private String userName;
 private String userStatus;


  public int getUserId() {
    return userId;
  }

  public void setUserId(int userId) {
    this.userId = userId;
  }

  public String getUserName() {
    return userName;
  }

  public void setUserName(String userName) {
    this.userName = userName;
  }

  public String getUserStatus() {
    return userStatus;
  }

  public void setUserStatus(String userStatus) {
    this.userStatus = userStatus;
  }
}
```  
2. 编写service和dao  
+ 在dao中进行数据库添加操作  
+ 调用JdbcTemplate对象的update方法实现数据库的增删改操作  
```java
public void add(Book book) {
//    jdbcTemplate.update(String sql,Object... args)
//    第一个参数是带有占位符的sql语句
//    第二个参数是占位符的参数值

//    也可以对可变参数使用数组传入
//    Objects[] args={book.getUserId(), book.getUserName(), book.getUserStatus()};
//    int update = jdbcTemplate.update(sql, args);

    String sql="insert into t_book values(?,?,?)";
    int update = jdbcTemplate.update(sql, book.getUserId(), book.getUserName(), book.getUserStatus());
    System.out.println(update);
  }
```








