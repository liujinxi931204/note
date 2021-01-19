## 什么是AOP  
面向切面编程，利用AOP可以对业务逻辑的各个部份进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发效率  
### AOP底层使用动态代理  
#### 有两种情况动态代理  
+ 有接口情况，使用JDK动态代理  
创建接口实现类代理对象，增强类方法  
+ 没有接口情况，使用CGLIB动态代理  
创建子类的代理对象，增强类方法  
#### JDK动态代理  
使用java.lang.reflect包下的Proxy类中的newProxyInstance(class.loader,类<?>{} interfaces,InvocationHandler h)方法  
##### 三个参数  
+ 第一个参数，类加载器  
+ 第二个参数，增强方法所在的类，这个类实现的接口，支持多个接口  
+ 第三个参数，实现这个接口InvocationHandler，创建代理对象，写增强方法  
```java
演示JDK动态代理

创建接口
public interface UserDao {

 public int add(int a,int b);

 public int update(int a);
}

创建接口的实现类
public class UserDaoImpl implements UserDao{


    @Override
    public int add(int a, int b) {
        return a+b;
    }

    @Override
    public int update(int a) {
        return a;
    }
}

使用Proxy类创建接口的代理对象
做增强的过程
public class UserDaoProxy {

    public static void main(String[] args) {

        Class[] interfaces={UserDao.class};
        UserDaoImpl userDaoImpl=new UserDaoImpl();

//创建接口实现类代理对象
        UserDao userDao = (UserDao)Proxy.newProxyInstance(UserDaoProxy.class.getClassLoader(),
                interfaces,
                new UserDaoProxyInvocation(userDaoImpl));
        userDao.add(1 ,2);
    }
}

//创建代理对象代码
class UserDaoProxyInvocation implements InvocationHandler{

//把被代理对象传入到代理类中取
//这里的被代理对象是UserDaoImpl，这里为了更加通用，所以使用Object
//要创建object的代理对象
    private Object object;

    public UserDaoProxyInvocation(Object object) {
        this.object = object;
    }

//这里实现增强的逻辑
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        System.out.println("方法执行之前..." + method.getName() + "传递的参数..." + Arrays.toString(args));
        Object invoke = method.invoke(object, args);
        System.out.println("方法执行之后...");
        return invoke;
    }
}


输出结果为
方法执行之前...add传递的参数...[1, 2]
方法执行之后...
```
### AOP操作术语  
+ 连接点  
类里面的哪些方法可以被增强，这些方法称为连接点  
+ 切入点  
实际被真正增强的方法，称为切入点  
+ 通知(增强)  
实际增强的逻辑部分称为通知  
通知由多种类型  
1. 前置通知
2. 后置通知  
3. 环绕通知
4. 异常通知  
5. 最终通知  
+ 切面  
把通知应用到切入点的过程称为通知  
### AOP操作  
+ Spring框架一般基于AspectJ实现AOP操作  
1. 什么是AspectJ不是Spring组成部分，独立于AOP框架，一般把AspectJ和Spring框架一起使用，进行AOP操作  
+ 基于AspectJ实现AOP操作  
1. 基于XML配置文件实现  
2. 基于注解方式实现  
+ 在项目工程中引入AOP相关依赖  
+ 切入点表达式  
1. 切入点表达式的作用：知道对哪个类里面的哪个方法进行增强  
2. 语法结构：  
```java
excution([权限修饰符] [返回类型] [全类名] [方法名称] ([参数列表]))
```
+ AOP操作(AspectJ注解) 
  
1. 创建被增强类，在类里面定义方法  
```java
//被增强的类
public class userDao {

    public void add(){
        System.out.println("add()...");
    }
}
```

2. 创建增强类(编写增强逻辑)  
在增强类中创建方法，让不同的方法代表不同的通知类型
```java
//增强的类
public class userDaoProxy {

    //前置通知
    public void before(){
        System.out.println("before()...");
    }
}
```

3. 进行通知的配置  
1）在spring的配置文件中，开启扫描注解  
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">


    <context:component-scan base-package="com.sogou"></context:component-scan>

</beans>
```
2）使用注解创建userDao和userDaoProxy对象  
```java
@Component("userDao")
public class userDao {

    public void add(){
        System.out.println("add()...");
    }
}

@Component("userDaoProxy")
public class userDaoProxy {

    public void before(){
        System.out.println("before()...");
    }
}
```
3）在增强类上面添加注解@Aspect  
```java
@Component("userDaoProxy")
@Aspect
//生成代理对象
public class userDaoProxy {

    public void before(){
        System.out.println("before()...");
    }
}
```
4) 在spring配置文件中开启生成代理对象  

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--开启注解扫描-->
    <context:component-scan base-package="com.sogou"></context:component-scan>
    <!--开启AspectJ生成代理对象-->
    <aop:aspectj-autoprox></aop:aspectj-autoprox>

</beans>
```

5. 配置不同类型的通知  
在增强类中，作为的通知方法上面添加配置类型注解，使用切入点表达式  
```java
@Component("userDaoProxy")
@Aspect
//生成代理对象
public class userDaoProxy {

    @Before("execution(* com.sogou..*.*(..))")
    public void before(){
        System.out.println("before()...");
    }
}
```

6. 测试  
```java
@Test
    public void testAop(){
        ApplicationContext applicationContext =
                new ClassPathXmlApplicationContext("applicationContext.xml");
        userDao userDao = applicationContext.getBean("userDao", userDao.class);
        userDao.add();
    }
输出结果为
before()...
add()...
```
## 公共切入点抽取  
将公共的切入点表达式提取出来，使用@Pointcut注解，还是上面的例子，只需要修改userDaoProxy.java  
```java
@Component("userDaoProxy")
@Aspect
//生成代理对象
public class userDaoProxy {
@Pointcut("execution(* com.sogou.spring.userDao.add(..))")
    public void pointDeom(){

    }

    @Before("pointDeom()")
    public void before(){
        System.out.println("before()...");
    }

}
```
这里将切入点表达式的公共部分("execution(* com.sogou.spring.userDao.add(..))")提取出来，使用@Pointcut注解，之后的前置通知可以简单的使用@Before("pointDeom")的方式，公共部分的提取也方便后续的更改  
## 有多个增强类对同一个方法进行增强，设置增强类的优先级  
+ 在增强类上添加注解@Order(数字类型值)，数字类型值越小优先级越高，最小是0  
例如，新增加一个personProxy.java，同样对add()方法做增强，可以通过@Order(数字类型)设置增强的优先级  
```java
@Component
@Aspect
@Order(1)
public class personProxy {

    @Before("execution(* com.sogou.spring..*.*(..))")
    public void before(){
        System.out.println("Person before()...");
    }
}

@Component("userDaoProxy")
@Aspect
@Order(3)
//生成代理对象
public class userDaoProxy {

    @Pointcut("execution(* com.sogou.spring.userDao.add(..))")
    public void pointDeom(){

    }


    @Before("pointDeom()")
    public void before(){
        System.out.println("before()...");
    }

}

测试方法
 @Test
    public void testAop(){
        ApplicationContext applicationContext =
                new ClassPathXmlApplicationContext("applicationContext.xml");
        userDao userDao = applicationContext.getBean("userDao", userDao.class);
        userDao.add();
    }

输出结果
Person before()...
before()...
add()...
```
## AOP操作(AspectJ配置文件)  
+ 创建两个类，增强类和被增强类，创建方法  
```java
被增强类
public class book {
    public void buy(){
        System.out.println("buy()...");
    }
}

增强类
public class bookProxy {

    public void before(){
        System.out.println("before()...");
    }
}

```
+ 在Spring配置文件中创建两个类对象  
```java
applicationContext.xml
 <bean id="book" class="com.sogou.SpringXML.book"></bean>
 <bean id="bookProxy" class="com.sogou.SpringXML.bookProxy"></bean>
```
+ 在Spring配置文件中配置切入点  
```java
<!--    配置aop增强-->
    <aop:config>
<!--        配置切入点，两个属性id和切入点表达式-->
        <aop:pointcut id="pointDemo" expression="execution(* com.sogou.SpringXML.book.buy(..))"/>

<!--        配置切面-->
        <aop:aspect ref="bookProxy">
<!--            配置增强作用在具体的方法上,这里意味着将before()方法作用在buy()方法上，并且是前置通知-->
            <aop:before method="before" pointcut-ref="pointDemo"></aop:before>
        </aop:aspect>
    </aop:config>

测试
 @Test
    public  void testBook(){
        ApplicationContext applicationContext =
                new ClassPathXmlApplicationContext("applicationContext.xml");

        book book = applicationContext.getBean("book", book.class);
        book.buy();
    }
输出结果为
before()...
buy()...
```









