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
1. PROPAGATION_REQUIRED  
+ 如果该方法执行在没有事务的方法中，就创建一个新的事务  
+ 如果执行在已经存在事务的方法中，则加入到这个事务中，合并成一个事务  

