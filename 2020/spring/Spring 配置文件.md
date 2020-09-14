## Bean实例化的三种方式  
+ 无参构造方法实例化  
+ 工厂静态方法实例化  
```java
配置文件applicationContxt.xml
 <bean id="user" class="com.sogou.spring.staticFactory" factory-method="getUser"></bean>
静态工厂
class staticFactory{
    public static User getUser(){
        return new User();
    }
}
实现类
class User{
    
}
测试方法
@Test
public void testUser(){
    ApplicationContext applicationContext =
                new ClassPathXmlApplicationContext("applicationContext.xml");
        User user = applicationContext.getBean("user", User.class);
        System.out.println(user);
}
输出结果为
com.sogou.spring.User@3c9754d8
```
+ 工厂实例方法实例化  
```java
配置文件applicationContxt.xml
 <bean id="factory" class="com.sogou.spring.staticFactory"></bean>
 <bean id="user" factory-bean="fatory" factory-method="getUser"></bean>
静态工厂
class staticFactory{
    public User getUser(){
        return new User();
    }
}
实现类
class User{
    
}
测试方法
@Test
public void testUser(){
    ApplicationContext applicationContext =
                new ClassPathXmlApplicationContext("applicationContext.xml");
        User user = applicationContext.getBean("user", User.class);
        System.out.println(user);
}
输出结果为
com.sogou.spring.User@3c9754d8
```  
  
**注意静态工厂的getUser()方法是静态的，可以直接在配置文件里使用factory-method,工厂方法的getUser()方法不是静态的，所以需要先获取工厂的Bean，再获取对象**   

## Bean标签基本配置  
用于创建对象交由Spring来创建  
默认情况下它调用的是类中的无参构造函数，如果没有无参构造函数则不能创建成功  
  
**基本属性**  
+ id：Bean实例在Spring容器中的唯一标识  
+ class：Bean的全限定名  
  
```java
<!--    配置User对象-->
    <bean id="user" class="com.sogou.spring.User"></bean>
<!--
```  
## Bean标签的范围标签  
+ scope：指对象的作用范围，取值如下  
singleton：默认值，单例的  
prototype：多例的  
  
+ 当scope的取值为singleton时：  
Bean的实例化个数：1个  
Bean的实例化时机：当Spring的核心文件被加载时，实例化配置的Bean实例  
Bean的生命周期  
对象创建：当应用加载，创建容器时，对象就被创建了  
对象运行：只要容器在，对象一直存活  
对象销毁：当应用加载，销毁容器时，对象就被销毁  
  
+ 当scope的取值为prototype时：
Bean的实例化个数：多个  
Bean的实例化时机：当调用getBean()方法时实例化Bean  
对象创建：当使用对象时，创建新的对象实例  
对象运行：只要对象在使用，就一直活着  
对象销毁：当对象长时间不用时，被java的垃圾回收器回收  


+ 单例  
```java
配置文件applicationContxt.xml
<!--    配置User单例对象-->
    <bean id="user" class="com.sogou.spring.User" scope="singleton"></bean>
<!--

测试方法  
@Test  
public void testSingleton(){
    ApplicationContext applicationContext = 
                new ClassPathXmlApplicationContext("applicationContext.xml");
    //加载配置文件，创建Spring容器
    //单例时，在加载配置文件的时候创建对象
    User user1=applicationContext.getBean("user",User.class);
    User user2=applicationContext.getBean("user",User.class);
    System.out.println(user1);
    System.out.println(user2);
}
结果输出为：
com.sogou.spring.User@3c9754d8
com.sogou.spring.User@3c9754d8  
说明是同一个对象
```  
或
```java
配置文件applicationContext.xml
<!--    配置User单例对象-->
    <bean id="user" class="com.sogou.spring.User"></bean>
<!--
测试方法  
@Test  
public void testSingleton(){
    ApplicationContext applicationContext = 
                new ClassPathXmlApplicationContext("applicationContext.xml");
    //加载配置文件，创建Spring容器
    //单例时，在加载配置文件的时候创建对象
    User user1=applicationContext.getBean("user",User.class);
    User user2=applicationContext.getBean("user",User.class);
    System.out.println(user1);
    System.out.println(user2);
}
结果输出为：
com.sogou.spring.User@3c9754d8
com.sogou.spring.User@3c9754d8  
说明是同一个对象
```
+ 多例  
```java
配置文件applicationContext.xml
<!--    配置User多例对象-->
    <bean id="user" class="com.sogou.spring.User" scope="prototype"></bean>
<!--
测试方法  
@Test  
public void testSingleton(){
    ApplicationContext applicationContext = 
                new ClassPathXmlApplicationContext("applicationContext.xml");
    //加载配置文件，创建Spring容器
    User user1=applicationContext.getBean("user",User.class);
    //多例的时候，在调用getBean()方法的时候创建对象，调用几次，创建几次
    User user2=applicationContext.getBean("user",User.class);
    System.out.println(user1);
    System.out.println(user2);
}
结果输出为：
com.sogou.spring.User@3c9754d8
com.sogou.spring.User@3bf7ca37 
说明是两个对象
```  
## Bean的依赖注入  
依赖注入(Dependency Injection):它是Spring框架核心IOC的具体实现  
+ set方法注入  
需要实现注入对象的set方法  
```java
配置文件applicatonContext.xml
   <bean id="user" class="com.sogou.spring.User">
       <property name="userName" value="AAA"></property>
   </bean>

实现类
class User{
    private String userName;

    public void setuserName(String userName){
        this.userName=userName;
    }
    
    @Override
    public String toString() {
        return "User{" +
                "userName='" + userName + '\'' +
                '}';
    }
}

测试

@Test
    public void testUser(){
        ApplicationContext applicationContext =
                new ClassPathXmlApplicationContext("applicationContext.xml");
        User user = applicationContext.getBean("user", User.class);
        System.out.println(user);
    }

输出结果为
User{userName='AAA'}
```  
+ P命名空间注入  
P命名空间注入本质也是set方法注入  
```java
首先引入p命名空间
配置文件applicationContext.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:p="http://www.springframework.org/schema/p"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

<!--  配置User类-->
<!--  id用来唯一标识这个对象-->
  <bean id="user" class="com.sogou.spring5.User"></bean>
</beans>

```

  
## Bean生命周期配置  
+ init-method:Bean的初始化方法  
+ destroy-method:Bean的销毁方法  
  
```java
//配置文件applicatonContxt.xml  
    <bean id="user" class="com.sogou.spring.User" init-method="initMethod" destroy-method="destroyMethod"></bean>
//类实现
class User{
    public void User(){
        System.out.println("User对象创建...");
    }

    public void initMethod(){
        System.out.println("初始化方法...");
    }

    public void destroyMethod(){
        System.out.println("销毁方法...");
    }
}
//测试
@Test
public void testUser(){
        ApplicationContext applicationContext =
                new ClassPathXmlApplicationContext("applicationContext.xml");
        User user = applicationContext.getBean("user", User.class);
        System.out.println(user);
        //ApplicatonContext是一个接口，没有实现close()方法
        //ClassPathXmlApplicationContext是其子类，功能更多，实现了close()方法
        //所以这里进行了强制类型转换
        ((ClassPathXmlApplicationContext)applicationContext).close();
}
结果输出为：  
创建对象...
初始化方法...
com.sogou.spring.User@4d1c00d0
销毁方法...
```  
  
+ Bean的生命周期：  
通过构造器创建Bean实例(无参构造)  
为Bean的属性设置值和其他Bean引用(调用set方法)  
调用Bean的初始化方法(需要配置上面的init-method)  
Bean可以使用了(对象获取到了)  
当容器关闭时候，调用Bean的销毁方法(需要配置上面的destroy-method)  
  
+ Bean的后置处理器，**调用Bean的后置处理器需要实现BeanPostProcessor接口**，Bean的生命周期有7步：  
通过构造函数创建对象(无参构造)  
为Bean的属性设置值和对其他Bean的引用调用set方法)  
**把Bean实例传递给Bean后置处理器的postProcessBeforeInitialization()**  
调用Bean的初始化方法(需要配置上面的init-method)  
**把Bean实例传递给Bean后置处理器的postProcessAfterInitialization()**  
Bean可以使用了(对象获取到了)  
当容器关闭的时候，调用Bean的销毁方法(需要配置上面的destroy-method)  
```java
配置文件applicationContext.xml 
    <bean id="user" class="com.sogou.spring.User" init-method="initMethod" destroy-method="destroyMethod">
        <property name="userName" value="AAA"></property>
    </bean>

    <bean id="postProcessor" class="com.sogou.spring.postProcessor"></bean>  

User类
public class User {

    private String userName;

    public User() {
        System.out.println("创建对象...");
    }

    public void initMethod(){
        System.out.println("初始化方法...");
    }

    public void destroyMethod(){
        System.out.println("销毁方法...");
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }
}

定义postProcessor，实现BeanPostProcessor接口
public class postProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("初始化之前...");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("初始化之后...");
        return bean;
    }
}

测试方法
 @Test
    public void testUser(){
        1.加载配置文件
        ApplicationContext applicationContext =
                new ClassPathXmlApplicationContext("applicationContxt.xml");
        // 2. 获取配置创建的对象
        User user = applicationContext.getBean("user", User.class);
        ((ClassPathXmlApplicationContext)applicationContext).close();

    }

结果输出为：
创建对象...
初始化之前...
初始化方法...
初始化之后...
销毁方法...
```
可以看到，Bean后置处理的postProcessBeforeInitialization()方法是在初始化方法之前执行的，postProcessAfterInitialization()是在初始化方法之后、获取到对象之前执行的  



