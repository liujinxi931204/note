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
Spring配置文件引入context命名空间  
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:p="http://www.springframework.org/schema/p"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
   http://www.springframework.org/schema/contexthttp://www.springframework.org/schema/context/spring-context.xsd">

<!--  配置连接池-->
  <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClass" value=""></property>
  </bean>

```