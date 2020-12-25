## ThreadLocal是什么  
ThreadLocal是JDK java.lang包中的一个用来实现不同线程的线程数据隔离的一个工具  
ThreadLocal这个类提供局部变量，这些变量与其他正常变量的不同之处在于，每一个访问该变量的线程在其内部都有一个独立的初始化的变量副本；ThreadLocal实例变量通常采用private static 在类中修饰  
只要ThreadLocal的变量能被访问，并且线程存活，那么每个线程都会持有ThreadLocal变量的副本。当一个线程结束的时候，它所持有的所有ThreadLocal相对应的实例副本都可被回收  
**ThreadLocal适用于每个线程需要自己独立的实例且该实例需要在多个方法中被使用(相同数据共享)，也就是变量在线程间隔离(不同的线程数据隔离)而在方法或类间共享的场景**  
    
**ThreadLocal的作用和同步机制有些相反：同步机制是为了保证多线程环境下数据的一致性；而ThreadLocal是保证了多线程环境下数据的独立性**  
## ThreadLocal类用在哪些场景  
一般来说，ThreadLocal在实际工业生产中并不常见，但是在很多框架中使用却能解决一些框架问题；比如如Spring中的事务、Spring中作用域Scope为Request的Bean使用ThreadLocal来解决  

## ThradLocal的特性  
ThreadLocal和synchornized都是为了解决多线程中相同变量的访问冲突问题，不同的点是  
+ synchornized是通过线程等待，牺牲时间来解决访问冲突  
+ ThreadLocal是通过每个线程单独一份存储空间，牺牲空间来解决冲突  
  
相比较于synchonized，ThreadLocal具有线程隔离的效果，只有在线程内才能获取到对应的值，线程外则不能访问到想要的值  
  
正因为ThreadLocal的线程隔离特性，所以它的应用场景更为特殊一些。当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，就可以考虑采用ThreadLocal实现  
## ThreadLocal的方法  
```java
//ThreadLocal是可以带泛型的

//get()方法是用来获取ThreadLocal在当前线程中保存的变量副本  
public T get();

//set()方法是用来设置当前线程中变量的副本
public void set(T value)；

//remove()方法是用来移除当前线程中变量的副本
public void remove();

//initialValue()是一个protected方法，一般是用来在使用时进行重写的，如果在没有set()的时候就是用get()，会调用initialValue()方法初始化内容  
protected T initvalValue();
```
 

