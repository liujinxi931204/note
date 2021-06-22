遇到多线程问题使用synchornized是一个百试不爽的良药。但是相比于j.u.c.Lock来说，synchornized是一个重量级的锁。不过随着对synchornized进行的各种优化，它也并不会显得那么重了  

## 基本使用  

synchornized是解决并发问题的一种最常用的方法，也是最简单的一种方法。synchornized主要有三个作用  

+ 原子性：保证线程互斥的访问同步代码  
+ 可见性：保证共享变量的修改能够及时可见，其实是通过java内存模型中的"**对一个变量unlock操作之前，必须要同步到主内存中；如果对一个变量进行lock操作，则将会清空工作内存中此变量的值，在执行引擎使用此变量前，需要重新从主内存中load操作或assign操作初始化变量值**"来保证的  
+ 有序性：有效解决重排序问题，即"一个unlock操作先行发生(happen-before)于后面对同一个锁的lock操作"  

先来看这样一段代码  

```java
public class LockTest
{
    Object obj=new Object();
    public static synchronized void testMethod1()
    {
        //同步代码。
    }
    public synchronized void testMethod2()
    {
        //同步代码
    }
    public void testMethod3()
    {
        synchronized (obj)
        {
            //同步代码
        }
    }
}
```

很多人都对synchornized的用法做了总结  

+ synchornized修饰静态方法（对应testMethod1），锁的是当前类的class对象，对应到这里就是LockTest.class对象  

+ synchornized修饰实例方法（对应testMethod2），锁的是当前类的实例对象，对应到这里就是LockTest中的this引用对象  

+ synchornized修饰同步代码块的时候（对应testMethod3），锁的是同步代码块括号里的对象实例，对应到这里就是obj对象  

