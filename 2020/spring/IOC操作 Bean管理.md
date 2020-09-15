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
//这里C
@Component(value = "user")//类似于<bean id="user">
public class User {

  @Override
  public String toString() {
    return "User{}";
  }
}
```



















 
