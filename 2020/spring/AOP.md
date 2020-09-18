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

