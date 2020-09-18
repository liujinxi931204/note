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
+ 第三个参数，实现这个接口InvocationHandler，拆
