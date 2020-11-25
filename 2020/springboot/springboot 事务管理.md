## 事务简介  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/11/25/1606286027591-1606286027661.png)  
### 事务是什么  
事务是一组原子操作单元，从数据库角度来说，就是一组SQL指令向数据库提交，要么全部执行成功，要么撤销不执行  
### 事务的四个属性  
事务概念包含四个特性(ACID)特性，主要有  
+ **原子性(Atomicity)**：原子性是指，这个事务当中执行的多个操作，它要么都执行完成，要么都不完成，它不会出现只完成其中一部分这种情况  
+ **一致性(Consistency)**：一致性是指，这个事务完成以后，它们的状态改变是一致的，它的结果是完整的。一致性更多的是从数据的状态或者结果来表现  
+ **隔离性(Isolation)**：隔离性是指，在执行不同的事务，它们视图操纵同样的数据的时候，它们之间是相互隔离的  
+ **持久性(Durability)**：持久性是指，当事务提交以后，数据操作的结果会存储到数据库中永久保存。如果事务还没有提交，就出现一些故障或者系统宕机等情况，导致事务没有提交，数据的修改不会出现在数据库当中  
### 什么是Spring事务  
Spring事务其实指的是Spring框架中的事务模块。在Spring框架中，对执行数据库事务的操作进行了一系列封装，其本质的实现还是在数据库，加入数据库不支持事务的话，Spring的事务也不会起作用，且Spring对事务进行了加强，添加了事务传播行为等功能  
## Spring事务抽象  
### Spring中事务核心接口类  
在Spring中核心的事务接口主要由以下组成  
+ **PlatformTransactionManager**：事务管理器  
+ **TransactionDefinition**：事务的一些基础属性定义，例如事务的传播属性、隔离级别、超时时间等  
+ **TransactionStatus**：事务的一些状态信息，如是否是一个新的事务、是否已被标记为回滚  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/11/25/1606286957023-1606286957025.png)  
### 事务管理器(PlatformTransactionManager)  
在Spring框架中并不直接管理事务，而是提供PlatformTransactionManager事务管理器接口类，对事务的概念进行抽象。它是将事务的实现交由其他持久层框架。例如Hibernate、Mybatis等都是实现了Spring事务的第三方持久层框架，由于每个框架中事务的实现各不相同，所以Spring对事务接口进行了统一，事务的提交、回滚等操作全部交由PlatformTransactionManager接口的实现类来进行实现  
### 隔离级别  
设置不同的事务隔离级别能解决不同的问题。在Spring中定义了五个事务隔离级别，每个隔离级别都有不同作用  
+ TransactionDefinition.ISOLATION_DEFAULT：使用数据库中配置的默认隔离级别  
+ TransactionDefinition.ISOLATION_READ_UNCOMMITTED：最低的隔离级别，允许读取已改变而没有提交的数据，可能会导致脏读、不可重复读和幻读  
+ TransacationDefinition.ISOLATION_READ_COMMITTED：允许读取事务已提交的数据，可以阻止脏读，但是可能会发生幻读、不可重复读  
+ TransactionDefinition.ISOLATION_READ_REPEATABLE_READ：对同一字段的多次读取结果都是一致的，除非数据事务本身改变，可以阻止脏读、不可重复读，但是可能会发生幻读  
+ TransactionDefinition.ISOLATION_SERIALIZABLE：最高的隔离级别，完全服从ACID的隔离级别，确保不会发生脏读、不可重复读和幻读，也是最慢的事务隔离级别，因为它通常是通过完全锁定事务相关的数据库表来实现的  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/11/25/1606288441306-1606288441307.png)  
## Spring事务传播行为  
在Spring中定义了7种事务传播行为，这种传播行为主要是为了解决事务方法调用事务或非事务方法时，如何处理事务的传播行为。在Spring中对事务的控制是通过AOP切面实现的，大部分都是通过使用@Transactional注解来使用事务  
### 传播机制生效条件  
因为Spring是使用AOP来实现事务控制的，是针对接口或者类的，所以在同一个类中的两个方法的调用，事务传播机制是不生效的  
+ 事务方法调用本类中方法(失效)  
+ 事务方法调用另一个类中的方法(有效)  
+ 非事务方法调用本类中的事务方法(无效)  
+ 非事务方法调用另一个类中的事务方法(有效)  

### 七种事务传播行为  
+ TransactionDefinition.PROPAGATION_REQUIRED（默认）：  
如果该方法执行在没有事务的方法中，就创建一个新的事务  
如果执行在已经存在事务的方法中，则加入这个事务中，合并成一个事务  
+ TransactionDefinition.PROPAGATION_SUPPORTS：  
如果该方法执行在没有事务的方法中，就以非事务方式执行  
如果执行在已经存在事务的方法中，则加入这个事务中，合并成一个事务  
+ TransactionDefinition.PROPAGATION_MANDATORY：  
如果该放没有执行在事务的方法中，就抛出异常  
如果执行在已经存在事务的方法中，则加入这个事务，合并成一个事务  
+ TransactionDefinition.PROPAGATION_REQUIRES_NEW：  
无论该方法是否执行在事务方法中，都创建一个新的事务  
不过如果执行在事务的方法中，就将方法中的事务暂时挂起  
新的事务会独立提交与回滚，不受调用它的父方法的事务影响  
+ TransactionDefinition.PROPAGATION_NOT_SUPPORTED：  
无论该方法是否执行在事务的方法中，都以非事务方式执行  
不过如果执行在存在事务的方法中，就将该事务暂时挂起  
+ TransactionDefinition.PROPAGATION_NEVER：
如果该方法执行在没有事务的方法中，也就以非事务方式执行  
不过如果执行在存在事务的方法中，就抛出异常  
+ TransactionDefinition.PROPAGATION_NESTED：
如果该方法执行在没有事务的方法中，就创建一个新的事务  
如果执行在已经存在的事务的方法中，则在当前事务中嵌套创建子事务执行  
被嵌套的事务可以独立于封装事务进行提交或者回滚  
如果外部事务提交嵌套事务也会被提交，如果外部事务回滚嵌套事务也会进行回滚  
**上述7中事务传播行为中比较常用的是REQUIRED、REQUIRES_NEW和NESTED**  
### 事务传播行为详解  
#### PROPAGATION_REQUIRED  
+ 如果该方法执行在没有事务的方法中，就创建一个新的事务  
+ 如果执行在已经存在事务的方法中，则加入到这个事务中，合并成一个事务  
```java
@Service
public class ServiceA{
   
    @Autowired
    private ServiceB serviceB;
    
    public void mA(){
   //业务逻辑
   serviceB.mB();
   }
}


@Service
public class ServiceB{
    
    @Transactional(propagation=propagation.REQUIRED)
    public void mB(){
   //业务逻辑

   }
}
```
1. 上面两个类中，只有ServiceB.mB方法设置了事务，且设置事务的传播行为是PROPAGATION_REQUIRED，当设置此传播行为时，当Service.mA()运行调用ServiceB.mB()时，ServiceB()发现自己执行在没有事务的ServiceA()方法中，这时ServiceB()会新建一个事务  
2. 上面两个类中的方法都设置了事务，且设置的事务传播行为是 PROPAGATION_REQUIRED，当设置此传播行为时，当 ServiceA.mA2() 运行调用 ServiceB.mB() 时，ServiceB.mB() 发现自己执行在已经存在 ServiceA.mA2() 设置的事务中，这时 ServiceB.mB() 不会再创建事务，而是直接加入到 ServiceA.mA2() 设置的事务中。这样，当 ServiceA.mA2() 或者 ServiceB.mB() 方法内发生异常时，两者都会回滚  
#### PROPAGATION_REQUIRED_NEW  
+ 无论该方法是否执行在事务的方法中，都会创建一个新的事务  
+ 不过如果执行在存在事务的方法中，就将方法中的事务暂时挂起  
+ 新的事务会独立提交和回滚，不受调用它的父方法的事务的影响  
```java
@Service
public class ServiceA {

    @Autowired
    private ServiceB serviceB;

    public void mA1() {
        // 调用另一个类中方法，测试事务传播行为
        serviceB.mB();
    }
    
    @Transactional
    public void mA2() {
        // 调用另一个类中方法，测试事务传播行为
        serviceB.mB();
    }
    
}

@Service
public class ServiceB {

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void mB() {
        System.out.println("方法 mB()");
        // 业务逻辑（略）
    }

}
```  
1. 上面方法中 ServiceA.mA1() 没有设置事务，而 ServiceB.mB() 设置了事务，且设置的事务行为是 PROPAGATION_REQUIRES_NEW。ServiceA.mA1() 运行调用 ServiceB.mB() 时，方法 ServiceB.mB() 发现调用自己的方法并没设置事务，这时方法 ServiceB.mB() 会创建一个新的事务。新事务中的提交与回滚不受调用它的父方法事务影响  
2. 上面方法中 ServiceA.mA2() 和 ServiceB.mB() 都存在事务，当 ServiceA.mA2() 运行调用 ServiceB.mB() 时，ServiceB.mB() 发现自己执行在已经存事务的方法中，这时 ServiceB.mB() 会创建一个新的事务，且将 ServiceA.mA2() 中的事务暂时挂起，等 ServiceB.mB() 完成后，恢复 ServiceA.mA2() 的事务。

由于 ServiceB.mB() 是新起一个事务，那么 ServiceA.mA2() 在调用 ServiceB.mB() 执行时 ServiceA.mA() 事务被挂起，那么：  
+ 假设 ServiceB.mB() 已经提交，那么 ServiceA.mA2() 抛出异常进行回滚，这时 ServiceB.mB() 是不会回滚的  
+ 假设 ServiceB.mB() 异常回滚，假设他抛出的异常被 ServiceA.mA2() 捕获，ServiceA.mA2() 事务仍然可能提交  
#### PROPAGATION_NESTED  
+ 如果该方法执行在没有事务的方法中，就创建一个新的事务  
+ 如果执行在已经存在事务的方法中，则当前事务中嵌套创建子事务执行  
+ 被嵌套的事务可以独立于封装事务进行提交或回滚  
+ 如果外部事务提交嵌套事务也会被提交，如果外部事务回滚嵌套事务也会进行回滚  
```java
@Service
public class ServiceA {

    @Autowired
    private ServiceB serviceB;

    public void mA1() {
        // 调用另一个类中方法，测试事务传播行为
        serviceB.mB();
    }
    
    @Transactional
    public void mA2() {
        // 调用另一个类中方法，测试事务传播行为
        serviceB.mB();
    }
    
}
@Service
public class ServiceB {

    @Transactional(propagation = Propagation.NESTED)
    public void mB() {
        System.out.println("方法 mB()");
        // 业务逻辑（略）
    }

}
```  
1. 上面方法中 ServiceA.mA1() 没有设置事务，而 ServiceB.mB() 设置了事务，且设置的事务行为是 PROPAGATION_NEVER。ServiceA.mA1() 运行调用 ServiceB.mB() 时，方法 ServiceB.mB() 发现调用自己的方法并没有设置事务，这时方法 ServiceB.mB() 就会创建一个新的事务  
2. 上面示例中两个方法都设置了事务，ServiceB.mB() 设置的事务行为是 PROPAGATION_NESTED。ServiceA.mA2() 运行调用 ServiceB.mB() 时，方法 ServiceB.mB() 发现调用自己方法也存在事务，这时方法 ServiceB.mB() 也会创建一个新的事务，与 ServiceA.mA2() 的事务形成嵌套事务。被嵌套的事务可以独立于封装事务进行提交或回滚。如果外部事务提交嵌套事务也会被提交，如果外部事务回滚嵌套事务也会进行回滚  
### 传播行为间的差异  
1. **ROPAGATION_REQUIRES_NEW 与 PROPAGATION_NESTED 的差异**  
传播行为**ROPAGATION_REQUIRES_NEW**和**PROPAGATION_NESTED**事务比较相似，很多时候很难分，这里简要说明下区别  
#### PROPAGARION_REQUIRES_NEW  
+ 设置PROPAGATION_REQUIRES_NEW时会创建一个"新事物",而不依赖于环境的"内部"事务  
+ 新事物的"提交"或"回滚"不依赖于外部事物，它拥有自己的隔离范围、锁等  
+ 当新的内部事务开始执行时，外部事务将被挂起，内事务结束时，外部事物将继续执行  
#### PROPAGATION_NESTED   
+ PROPAGATION_NESTED开始一个"嵌套事务",它是已经存在事务的一个真正的子事务  
+ 嵌套事务开始执行时，它将取得一个savepoint回滚点，如果这个嵌套事务失败，将会回滚到这个回滚点  
+ 嵌套事务是外部事物的一部分，只有外部事物结束后它才会被提交  
2. 事务的差异总结  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/11/25/1606296185903-1606296185905.png)  
## Spring事务的两种实现  
### 编程式事务和声明式事务  
Spring支持编程式事务管理和声明式事务管理两种方式  
+ **编程式事务**:编程式事务使用TransactionTemplate或者直接使用底层的PlatformTransactioinManager实现事务。对于编程式事务Spring比较推荐使用TransactionTemplate来对事务进行管理  
+ **声明式事务**:声明式事务是建立在AOP只上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况"提交"或"回滚"事务  
### 两种事务管理间的区别  
+ 编程式事务允许用户在代码中精确定义事务的边界  
+ 声明式事务有助于用户将操作与事务规则进行解耦，它是基于AOP交由Spring容器实现  
  
编程式事务其侵入到了业务代码里，但是提供了更加纤细的事务管理。而声明式事务基于AOP，所以既能起到事务作用，又可以不影响业务代码的具体实现。一般而言比较推荐使用声明式事务，尤其是@Transactional注解，









