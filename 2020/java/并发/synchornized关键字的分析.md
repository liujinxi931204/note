遇到多线程问题使用synchronized是一个百试不爽的良药。但是相比于j.u.c.Lock来说，synchronized是一个重量级的锁。不过随着对synchronized进行的各种优化，它也并不会显得那么重了  

## 基本使用  

synchronized是解决并发问题的一种最常用的方法，也是最简单的一种方法。synchronized主要有三个作用  

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

很多人都对synchronized的用法做了总结  

+ synchronized修饰静态方法（对应testMethod1），锁的是当前类的class对象，对应到这里就是LockTest.class对象  

+ synchronized修饰实例方法（对应testMethod2），锁的是当前类的实例对象，对应到这里就是LockTest中的this引用对象  

+ synchronized修饰同步代码块的时候（对应testMethod3），锁的是同步代码块括号里的对象实例，对应到这里就是obj对象  

从这里可以看到，synchronized的使用都要依赖特定的对象  

## 同步原理  

数据同步需要依赖锁，那锁的同步又依赖谁？**synchronized给出的答案是在软件层面依赖JVM，而j.u.c.Lock给出的答案是在硬件层面依赖特殊的CPU指令**  

当一个线程访问同步代码块时，首先需要得到锁才能执行同步代码，当退出或者抛出异常时必须要释放锁，那么它是如何来实现这个机制呢？先看一段简单的代码  

```java
public class SynchronizedDemo {
    public void method() {
        synchronized (this) {
            System.out.println("Method 1 start");
        }
    }
}
```

查看反编译后的结果  

![img](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/2062729-b98084591219da8c.png)

1. monitorenter：每个对象都是一个监视器锁（monitor）。当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权，过程如下  

   + 如果monitor的进入数为0，则该线程进入monitor，然后将monitor的进入数设置为1，该线程即为monitor的所有者  

   + 如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1  

   + 如果其他线程已经占有了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权  

2. mointorexit：执行monitorexit的线程必须式monitor的所有者。执行指令时，monitor的进入数减1，如果减1后为0，那线程退出monitor，不再是这个monitor的所有者。其他被这个monitor阻塞的线程可以尝试去获取这个monitor的所有权  

   + 在上面反编译后的代码中monitorexit出现了两次，第一次为正常退出释放锁，第二次为异常退出释放锁  

通过上面的两端描述，可以很清楚的看出synchronized的实现原理，synchronized的语义底层是通过一个monitor对象来完成的，其实wait/notify等方法的底层也依赖于monitor对象，这就是为什么只有在同步的块或者方法中才能调用wait/notofy等方法，否则会抛出java.lang.IllegalMonitorStateException异常的原因。  

再来看一同步方法  

```java
public class SynchronizedMethod {
    public synchronized void method() {
        System.out.println("Hello World!");
    }
}
```

查看反编译后的结果  

![img](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/2062729-8b7734120fae6645.png)

从编译的结果来看，方法的同步没有通过指令monitorenter和monitorexit来完成，不过相对于普通方法，其常量池中多了ACC_SYNCHRONIZED标识符。JVM就是根据该标识符来实现同步方法的：  

+ 当方法调用时，调用指令将会检查方法的ACC_SYNCHRONIZED访问标识符是否被设置了，如果设置了，执行线程将会先获取monitor，获取成功后才能执行方法体，方法执行完后再释放monitor。再方法执行期间，其他任何线程都无法再获得monitor对象  

两种同步方法本质上没有区别，只是方法的同步是一种隐式的方式来实现，无需通过字节码来完成。两个指令的执行是JVM通过调用操作系统的互斥原语mutex来实现，被阻塞的线程会被挂起，等待重新调度，会导致"用户态和内核态"之间的切换，对性能有较大的影响。这也就是为什么synchronized早期被称为重量级锁的原因。  

## 对象的组成  

java中一切皆为对象，对象由三部分组成，分别是对象头、示例数据、对齐填充  

实例数据就是在类中定义的那些字段数据所占用的空间。对齐填充是因为java虚拟机要求对象的大小必须是8字节的整数倍，如果一个对象所占用的存储空间不是8字节的整数倍，需要把它填充到8字节的整数倍。而与锁有关的则是对象头  

![对象组成](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E5%AF%B9%E8%B1%A1%E7%BB%84%E6%88%90.png)  

以32位虚拟机为例，Mark Word有四个字节，而且还要存放Hash Code等信息，难道锁就可以由这四个字节来实现吗？  

![在这里插入图片描述](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/20200619123714116.png)  

## 锁升级的过程  

在JDK 1.6时，虚拟机团队对synchronized进行了一系列的优化，其中一项重要的优化就是锁升级  

synchronized的锁升级过程：由无锁升级为偏向锁，再升级为轻量级锁，最后升级为重量级锁，这一过程是不可逆的  

也就是说，除非存在很严重的线程竞争，否则synchronized不会直接使用重量级锁了  

 我们知道现实世界中是由我们人来负责进行上锁和开锁的，那么程序世界中其实是由线程来扮演人的角色来进行加锁解锁的。  

### 偏向锁  

刚开始的时候，处于无锁状态，这时第一个线程运行到了同步代码区域，加上了一个偏向锁，这时偏向锁是一种什么状态？其实就是将能够唯一标识线程的线程ID保存到了Mark Word中  

![偏向锁](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E5%81%8F%E5%90%91%E9%94%81.png) 

这里的4个字节的23位用来存储代表第一个获取偏向锁的线程的线程ID，2位Epoch代表偏向锁的有效性，4位对象分代年龄，1位是否偏向（1为是），2位锁标志位（01是偏向锁）  

当第一个线程运行到同步代码块的时候，会去检查synchronized锁使用的那个对象的对象头，如果上面说的synchronized所使用的对象头的线程ID为空的化，并且偏向锁是有效的，说明当前处于无锁状态。那么这时候第一个线程就会将自己的线程ID替换到对象头的Mark Word的线程ID，如果替换成了说明该线程获得了偏向锁，那么线程就可以安全的执行同步代码块了。以后如果线程再次进入同步代码块的时候，再次期间如果没有其他线程获取偏向锁，只需要简单对比一下自己的线程ID和Mark Work中的线程ID是否一致，如果一致就可以直接进入同步代码区域，这也性能损耗就小很多了  

### 轻量级锁  

偏向锁是在不存在多个线程竞争锁的情况下存在的，然后高并发的情况下竞争锁是不可避免的，此时synchronized开启了它的晋升之路  

锁的升级大概可以分为两步。1. 偏向锁的撤销；2. 轻量级锁的升级  

首先偏向锁是如何撤销的呢？偏向锁其实就是Mark Word中的线程ID，这个时候只要更改Mark Word自然就相当于撤销了偏向锁。那么问题在于偏向锁是用线程ID表示的，轻量级锁用什么表示呢？答案是Lock Record（栈帧中的锁记录） 。 

我们知道JVM内存结构分为堆、虚拟机栈、本地方法栈、程序计数器、方法区、直接内存。这其中程序计数器和虚拟机栈是线程私有的，每个线程拥有自己的独立的栈空间，看起来存放在栈中的数据可以很好的区分是哪个线程获取到了锁。  

首先，JVM会在当前的栈中开辟一块内存，这块内存称为Lock Record（锁记录），并把Mark Word中内容复制到Lock Record中，即在Lock Record中存放的是之前的Mark Word中的内容，方便日后的恢复。  然后更新对象头中的Mark Word的内容为指向Lock Record中指针。Mark Word中的内容如下 ： 

![轻量级锁](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E8%BD%BB%E9%87%8F%E7%BA%A7%E9%94%81.png)  

可以看到Mark Word中使用30位来记录刚刚在栈帧中创建的Lock Record，锁标志位为00表示轻量级锁，这也就很容知道哪个线程获取了轻量级锁。

轻量级锁是基于这也一个事实，当存在两个或以上的线程竞争锁的时候，绝大多数情况下，持有锁的线程是会很快释放锁的，也就是当锁存在少量竞争时，通常情况下锁被持有的时间很短，此时等待锁的线程不必进行用户态和内核态的切换，而只要自旋一会，希望在自旋的这段时间内锁被释放。很明显，轻量级锁适用于竞争不是很激烈而且锁被持有的时间非常短的情况，相反，如果锁的竞争很激烈或者锁被持有的时候很长，那么线程会白白地自旋而浪费CPU资源。  

### 重量级锁  

JVM设定了一个自旋的限制，如果线程自旋了一定的次数之后仍然没有获取到锁，就可以视为锁竞争比较激烈的情况，这时轻量级锁就要晋升为重量级锁。  

重量级锁的复杂度是最高的，由于持有锁的线程在释放时候需要唤醒阻塞等待的线程，线程获取不到锁的时候需要进入某一个阻塞区域统一阻塞等待，同时还有wait、notify条件的等待与唤醒需要处理，所以重量级锁的实现需要以一个额外的Monitor。  

以HotSpot虚拟机为例，虚拟机中对Monitor的实现使用ObjectMonitor实现，这两者的关系类似于java中的Map和HashMap的关系。    

```c++
 ObjectMonitor() 
  {
    _header       = NULL;
    _count        = 0;//用来记录该线程获取锁的次数
    _waiters      = 0,
    _recursions   = 0;//锁的重入次数
    _object       = NULL;
    _owner        = NULL;//指向持有ObjectMonitor的线程
    _WaitSet      = NULL;//存放处于Wait状态的线程的集合
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ;//所以等待获取锁而被阻塞的线程的集合
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
  }
```

首先ObjectMonitor中需要有一个指针指向当前获取锁的线程，就是上面ObjectMonitor中的owner，当某一个线程获取锁的时候，将调用ObjectMonitor.enter()方法进入同步代码块，获取到锁之后，将owner设置为指向当前线程，当其他的线程尝试获取锁的时候，就找到ObjectMonitor的owner看看是否是自己。如果是的话，recursions和count自增1，代表该线程再次获取到了锁（synchronized是可重入锁，持有锁的线程可以再次的获取锁），否则的话就应该阻塞起来，那么这些阻塞的线程放在哪里呢？  

统一的放在EntryList中即可。当持有锁的线程调用wait方法时（wait方法会使得线程放弃cpu并释放自己持有的锁，然后阻塞挂起自己，直到其他的线程调用notify或者notifyAll方法为止），那么线程该释放掉锁，把owner置空，并唤醒在EntryList中阻塞等待获取锁的线程，然后将自己挂起并进入waitSet集合中等待。当其他持有锁的线程调用了notify或者notifyAll方法时，会将waitSet中的某一个线程或者全部线程唤醒，从waitSet中移动到EntryList中等待竞争锁。当线程要释放锁的时候，就会调用ObjectMonitor.exit()方法退出同步代码块。  

![img](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/2062729-d1cc81ebcf0e912b.png)  

要撤销轻量级锁，当然要把保存在栈帧中的LockRecord中存储的内容在写回到Mark Word中，然后将栈帧中的Lock Record清理掉。此后需要创建一个ObjectMonitor对象，并将Mark Word中的内容保存到ObjectMonitor中，更新对象头中的Mark Word指向这个ObjectMonitor对象。  

![重量级锁](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E9%87%8D%E9%87%8F%E7%BA%A7%E9%94%81.png)  

可以看到Mark Word中使用30位来保存指向ObjectMonitor的指针，锁标志位为10表示重量级锁  

## 锁形态的变迁  

当锁以偏向锁存在时，锁就是Mark Word中的线程ID，此时线程本身就是打开锁的钥匙，Mark Word中存了哪个线程的线程ID，哪个线程就获取到了锁。  

当锁以轻量级锁存在的时候，锁就是Mark Word中所指向栈帧中锁记录的Lock Record，此时哪个线程的栈帧中有Lock Record，哪个线程就获取到了锁。

当锁以重量级锁存在时候，锁就是c++中对于Monitor的实现ObjectMonitor，此时ObjectMonitor中的owner指向了哪个线程，哪个线程就获取到了锁。  
