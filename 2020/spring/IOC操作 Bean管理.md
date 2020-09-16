## 自动装配(基于xml)  
根据指定装配规则(属性名或者属性类型)，Spring自动将匹配的属性值进行注入  
实际中很少用到基于xml的自动装配，用的更多的是基于注解的自动装配  
  
自动装配的两种形式  
+ 根据属性名称来自动装配  
```java
配置文件applicationContext.xml
<!--
    bean标签的autowire属性实现自动装配
    autowire两个属性值：
        byName根据属性名进行注入，注入bean的id值和类属性名一致
-->

  <bean id="emp" class="com.sogou.spring5.Emp" autowire="byName"></bean>
  <bean id="dept" class="com.sogou.spring5.Dept"></bean>

Emp类实现
public class Emp {

  private Dept dept;

  public void setDept(Dept dept) {
    this.dept = dept;
  }

  @Override
  public String toString() {
    return "Emp{" +
        "dept=" + dept +
        '}';
  }
}

Dept类实现
public class Dept {

  @Override
  public String toString() {
    return "Dept{}";
  }
}

测试类
@Test
  public void testEmp(){
    ApplicationContext applicationContext =
        new ClassPathXmlApplicationContext("applicationContext.xml");

    Emp emp = applicationContext.getBean("emp", Emp.class);

    System.out.println(emp);

  }
输出结果为:
Emp{dept=Dept{}}
```
+ 根据属性类型来自动装配  
```java
配置文件applicationContext.xml
<!--
    bean标签的autowire属性实现自动装配
    autowire两个属性值：
        byType根据属性类型进行注入，相同类型的bean不能定义多个
-->

  <bean id="emp" class="com.sogou.spring5.Emp" autowire="byType"></bean>
  <bean id="dept" class="com.sogou.spring5.Dept"></bean>

Emp类实现
public class Emp {

  private Dept dept;

  public void setDept(Dept dept) {
    this.dept = dept;
  }

  @Override
  public String toString() {
    return "Emp{" +
        "dept=" + dept +
        '}';
  }
}

Dept类实现
public class Dept {

  @Override
  public String toString() {
    return "Dept{}";
  }
}

测试类
@Test
  public void testEmp(){
    ApplicationContext applicationContext =
        new ClassPathXmlApplicationContext("applicationContext.xml");

    Emp emp = applicationContext.getBean("emp", Emp.class);

    System.out.println(emp);

  }
输出结果为:
Emp{dept=Dept{}}
```
## 外部属性文件(数据库连接池为例)  
```java

创建外部属性文件，properties格式文件，写数据库信息jdbc.properties
prop.driverClass=com.mysql.jdbc.Driver
prop.url=jdbc:mysql://localhost:3306/userDB
prop.userName=root
prop.password=root


Spring配置文件引入context命名空间,通过context命名空间，把外部properties配置文件Spring配置文件中  
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
   http://www.springframework.org/schema/contexthttp://www.springframework.org/schema/context/spring-context.xsd">

在Spring配置文件中使用标签引入外部属性文件
<!--引入外部属性-->
<context:property-placeholder location="classpath:jdbc.properties"/>

<!--  配置连接池,使用jdbc.properties的key-->
  <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="${prop.driverClass}"></property>
    <property name="url" value="${prop.url}"></property>
    <property name="username" value="${prop.userName}"></property>
    <property name="password" value="${prop.password}"></property>
  </bean>

```  
## Bean管理(注解)  
### 什么是注解  
注解是代码的一种特殊标记，格式：@注解名称(属性名称=属性值，属性名称=属性值...)  
### 哪些地方可以使用注解  
注解作用在类上、方法上、属性上  
### Spring使用注解的目的  
简化xml配置   
### Spring针对Bean管理提供中创建对象提供注解  
+ @Component  
+ @Service  
+ @Controller  
+ Repository  
**上面四个注解功能都是一样的，都可以用来创建Bean的实例**  
### 基于注解方式实现对象的创建  
+ 引入依赖  
引入aop的jar包  

+ 开启组件扫描，让Spring知道扫描哪些类的注解
```java
引入context命名空间
配置文件applicatonContext.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                     http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

<!--
   开启组件扫描，会对base-package下的所有类的组件进行扫描
   如果有多个包，可以用逗号分隔开，也可以写多个包的共同的上级
-->
<context:component-scan base-package="com.sogou.spring"></context:component-scan>
</beans>
```  
+ 创建类和测试类  
```java
实现类
//这里Component后面可以省略，如果省略value的值就是类名，但是首字母小写
//如果不省略，就是value配置的值
//这个值的作用类似于xml配置里的bean id
//这里使用上面四个任意一个注解都是一样的
@Component(value = "user")//类似于<bean id="user">
public class User {

  @Override
  public String toString() {
    return "User{}";
  }
}

测试类
@Test
  public void testUser(){
    ApplicationContext applicationContext =
        new ClassPathXmlApplicationContext("applicationContext.xml");

    User user = applicationContext.getBean("user", User.class);
//这里getBean()中的就是上面@Component注解的value值
    System.out.println(user);
  }

输出的结果为
User{}
```  
+ 两个开启组件扫描的细节  
```java
<!--
    Spring组件扫描的时候会使用默认的过滤器，当设置use-default-filters="false"表示不使用默认的过滤器此时需要自己配置
    context:include-filter用来设置扫描哪些内容
    所以下面的例子就是扫面包com.sogou.spring下所有注解为Component的类，其余的注解不扫描
-->
  <context:component-scan base-package="com.sogou.spring" use-default-filters="false">
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Component"/>
  </context:component-scan>


  <!--
    context:exclude-filter用来设置不扫描哪些内容
    所以下面的例子就是扫面包com.sogou.spring下所有除了注解为Component的类，其余的注解全扫描
-->
  
  <context:component-scan base-package="com.sogou.spring">
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Component"/>
  </context:component-scan>
```  
### 基于注解方式的依赖注入  
+ @value 注入普通类型的属性  
+ @Autowired 根据类型自动注入  
例如  
```java
定义UserDao接口
public interface UserDao {

  public void addDao();

}

实现接口
@Repository
//默认值就是类型名，但是首字母需要小写，即userDaoImpl
public class UserDaoImpl implements UserDao {

  @Override
  public void addDao() {

    System.out.println("UserDaoImpl add()...");
  }
}

接口的实现作为另一个类的属性
@Service
public class UserService {
  @Autowired
  //不需要set()方法
  private UserDao userDao;

  public void add(){
    System.out.println("UserService add()...");
    userDao.addDao();
  }

}


测试
@Test
  public void testUserService(){
    ApplicationContext applicationContext =
        new ClassPathXmlApplicationContext("applicationContext.xml");

    UserService userService = applicationContext.getBean("userService", UserService.class);
    userService.add();
  }
输出结果为
UserService add()...
UserDaoImpl add()...
```
+ @Qualifier 根据类型名进行注入，和@Autowire搭配使用  
可以用在接口有多个实现类的时候，需要使用@Qualifier(属性名)来确认使用哪一个实现来进行注入  
```java
定义UserDao接口
public interface UserDao {

  public void addDao();

}

接口的具体实现类
@Repository("userDaoImpl")
//默认值就是类型名，但是首字母需要小写，即userDaoImpl
public class UserDaoImpl implements UserDao {

  @Override
  public void addDao() {

    System.out.println("UserDaoImpl add()...");
  }
}


```

+ @Resource 可以根据类型也可以根据类型名注入



















 
