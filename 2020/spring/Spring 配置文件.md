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
+ scope：指对象的作用范围，