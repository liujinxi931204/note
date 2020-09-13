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
  
## Bean生命周期配置  
+ init-method:Bean的初始化方法  
+ detroy-method:Bean的销毁方法  
  
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
