AQS（AbstarctQueuedSynchronized）是java众多锁以及并发工具的基础，其底层采用乐观锁，大量使用了CAS操作，并且在冲突时，采用自旋方式重试，以实现轻量级和高效地获取锁。  

AQS虽然被定义为抽象类，但事实上它并不包含任何抽象方法。这是因为AQS是被用来设计支持多种用途的，如果定义抽象方法，则子类继承的时候必须重写所有的抽象方法，这就有些不合理了。所以AQS将一些需要子类重写的方法都设计成protect方法，将其默认实现为抛出UnsupportedOperationException异常。如果子类使用到这些方法而没有重写，则会抛出异常；如果子类没有用到这些方法，则不需要做任何操作。  

AQS实现了锁的获取框架，锁的实际获取逻辑则交由子类去实现，就锁的获取而言，子类必须重写tryAcquire()方法。  

## java并发工具类的三个要点  

java并发工具需要抓住三个要点：  

+ 状态：一般是一个state属性，它是整个工具的核心，通常整个工具都是在设置和修改状态，很多方法的操作都依赖于当前状态是什么。由于状态是全局共享的，一般会被设置成volatile类型，以保证其修改的可见性  

+ 队列：队列通常是一个等待的集合，大多以链表的形式实现。队列采用的是悲观锁的思想，表示当前所等待的资源、状态或者条件短时间内可能无法满足。因此，它会将当前线程包装成一某种类型的数据结构，放到一个等待队列中，当一定条件满足后，再从队列中取出  

+ CAS：CAS操作时最轻量的并发处理，通常对于状态的修改都会用到CAS操作。这是因为状态可能会被多个线程同时修改，CAS操作保证了同一时刻只有一个线程可以修改成功，因此保证了线程安全。CAS操作基本时有Unsafe工具类的compareAndSetXXX来实现的。CAS采用的是乐观锁的思想，因此常常伴随自旋，其表现形式为   

  ```java
  for(;;){
      //自旋操作
  }
  ```

## AQS核心实现  

### 状态  

在AQS中，状态是由state属性来表示  

```java
	/**
     * The synchronization state.
     */
    private volatile int state;
```

该属性的值表示了锁的状态，state为0表示锁没有被占用，state大于0表示当前已经有线程持有该锁，这里说大于0而不是等于0，是因为存在可重入的情况。当大于0的时候，表示该锁被重入的次数。  

可是只有一个state属性，仅仅能知道这个锁被占用，但是被谁占用还不知道。因此有另外一个属性exclusiveOwnerThread来表示当前锁的持有者

```java
//exclusiveOwnerThread不是AbstarctQueuedSynchronized抽象类的属性，而是由它的子类的属性
private transient Thread exclusiveOwnerThread;
```

### 队列  

AQS中，队列的实现是一个双向链表，它表示所有等待锁的线程的集合，有点类似于synchronized中的entry set。

在前面说过，AQS会把等待锁的线程包装成某一种数据结构放入队列中。这个数据结构如下  

```java
static final class Node {
    /** Marker to indicate a node is waiting in shared mode */
    static final Node SHARED = new Node();
    /** Marker to indicate a node is waiting in exclusive mode */
    static final Node EXCLUSIVE = null;

    /** waitStatus value to indicate thread has cancelled */
    static final int CANCELLED =  1;
    /** waitStatus value to indicate successor's thread needs unparking */
    static final int SIGNAL    = -1;
    /** waitStatus value to indicate thread is waiting on condition */
    static final int CONDITION = -2;
    /**
     * waitStatus value to indicate the next acquireShared should
     * unconditionally propagate
     */
    static final int PROPAGATE = -3;

    volatile int waitStatus;

    volatile Node prev;

    volatile Node next;

    volatile Thread thread;

    Node nextWaiter;

    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    Node() {    // Used to establish initial head or SHARED marker
    }

    Node(Thread thread, Node mode) {     // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }

    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```

这个结构的属性主要有4类  

```java
// 节点所代表的线程
volatile Thread thread;

// 双向链表，每个节点需要保存自己的前驱节点和后继节点的引用
volatile Node prev;
volatile Node next;

// 线程所处的等待锁的状态，初始化时，该值为0
volatile int waitStatus;
static final int CANCELLED =  1;
static final int SIGNAL    = -1;
static final int CONDITION = -2;
static final int PROPAGATE = -3;

// 该属性用于条件队列或者共享锁
Node nextWaiter;
```

这里有一个waitStatus属性，表示了当前Node等待锁的状态，在独占锁的模式行，只需要关心CANCELLED和SIGNAL两种状态即可。还有一个nextWaiter属性，在独占锁的模式下，永远为null。  

说完队列中的节点数据结构以后，再来看一下这个双向链表。  

```java
// 头结点，不代表任何线程，是一个空结点
private transient volatile Node head;

// 尾节点，每一个请求锁的线程会加到队尾
private transient volatile Node tail;
```

在AQS中的队列是一个CLH队列，它的头节点是一个空节点（dummy node），它不代表任何线程（某些情况下可以看作是代表了持有锁的当前线程），因此head所指向的Node的thread永远都是null。也就是说，只有从头节点往后的节点才代表了所有等待所锁的线程。

![CLH队列](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/CLH%E9%98%9F%E5%88%97.png)

再来看一下Node节点各个参数的意义  

+ thread：表示当前Node所代表的线程  

+ waitStatus：表示节点所处的状态，独占锁模式下只需要关注 CANCELLED、SIGNAL和初始化  

+ prev、next：表示节点的前去和后继  

+ nextWaiter：独占锁模式下为null  

## AQS核心属性  

1. 锁相关的属性有两个：  

   ```java
   private volatile int state; //锁的状态
   private transient Thread exclusiveOwnerThread; // 当前持有锁的线程，注意这个属性是从AbstractOwnableSynchronizer继承而来
   ```

2. CLH队列相关的属性有两个  

   ```java
   private transient volatile Node head; // 队头，为dummy node
   private transient volatile Node tail; // 队尾，新入队的节点
   ```

3. 队列中Node节点需要关注的属性有三组  

   ```java
   // 节点所代表的线程
   volatile Thread thread;
   
   // 双向链表，每个节点需要保存自己的前驱节点和后继节点的引用
   volatile Node prev;
   volatile Node next;
   
   // 线程所处的等待锁的状态，初始化时，该值为0
   volatile int waitStatus;
   static final int CANCELLED =  1;
   static final int SIGNAL    = -1;
   ```

## ReentrantLock 

前面提到过，AQS大多数情况下都是通过继承来使用的，子类通过重写tryAcquire()方法来实现自己获取锁的逻辑。  

ReentrantLock分为公平锁和非公平锁两种实现，默认实现为非公平锁  

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    /** Synchronizer providing all implementation mechanics */
    private final Sync sync;
    
    /**
     * Base of synchronization control for this lock. Subclassed
     * into fair and nonfair versions below. Uses AQS state to
     * represent the number of holds on the lock.
     */
    abstract static class Sync extends AbstractQueuedSynchronizer {
        ...
    }
    
    /**
     * Sync object for non-fair locks
     */
    static final class NonfairSync extends Sync{
        ...
    }
    
    /**
     * Sync object for fair locks
     */
    static final class FairSync extends Sync {
        ...
    }
    
    /**
     * Creates an instance of {@code ReentrantLock}.
     * This is equivalent to using {@code ReentrantLock(false)}.
     */
    public ReentrantLock() {
        sync = new NonfairSync();
    }

    /**
     * Creates an instance of {@code ReentrantLock} with the
     * given fairness policy.
     *
     * @param fair {@code true} if this lock should use a fair ordering policy
     */
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
    
    // 获取锁
    public void lock() {
        sync.lock();
    }
    
    ...
}
```

由上面可以知道，ReentrantLock有一个内部类Sync，该类继承了AbstractQueuedSynchronizer抽象类，ReentrantLock另有两个内部类FairSync（公平锁）和NonFairSync（非公平锁）均继承了Sync这个内部类。默认情况下，ReentrantLock是非公平锁。

一般情况下ReentrantLock的使用格式如下  

```java
Lock lock = new ReentrantLock();
...
lock.lock();
try {
    // 更新对象
    //捕获异常
} finally {
    lock.unlock();
}
```

## 获取锁  

来看一下ReentrantLock的lock()方法：  

```java
public void lock() {
    sync.lock();
}
```

ReentrantLock的lock()方法调用了其内部类Sync的lock方法。Sync中lock方法是一个抽象方法，需要由各个实现类自己实现具体的逻辑。 FairSync和NonFairSync的lock方法的实现略有区别。具体如下：   

在公平锁FairLock中  

```java
final void lock() {
    acquire(1);
}
```

  在非公平锁NonFairLock中  

```java
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

可以看到无论公平锁还是非公平锁最后都使用了acquire的方法。  

### acquire方法  

acquire定义在AQS类中，描述了获取锁的流程  

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

acquire方法的顺序是首先执行tryAcquire方法，如果成功，则表示获取锁成功，退出acquire方法；如果失败，继续执行addWaiter方法，接着执行acquireQueued方法；如果acquireQueued方法执行成功，退出acquire方法；如果失败了，则会执行selfInterrupt方法，最后退出acquire方法。

可以看出，该方法涉及了四方法的调用  

+ tryAcquire(arg)  
  该方法有继承了AQS的子类实现，为获取锁的具体逻辑。在ReentrantLock中，就是由FairSync和NonFairSync分别实现了tryAcquire的具体逻辑  

+ addWaiter(Node mode)  

  该方法由AQS负责实现，在所获取失败后调用。主要就是将当前请求锁的线程包装成一个Node节点，添加到CLH对立中，并返回这个Node  

+ acquireQueued(final Node node,int arg)  

  该方法由AQS实现，这个方法比较复杂，主要对上面刚加入队列的Node不断尝试如下的操作  
  
  1. 在前驱节点就是head节点的时候，继续尝试获取锁  
  
  2. 将当前线程挂起，使CPU不再调度它  
  
+ selfInterrupt  
  
  该方法由AQS实现，用于中断当前线程。由于在整个枪锁的过程中是不响应中断的。那如果在抢锁的过程中发生了中断怎么办？AQS的做法就是简单的记录有没有发生过中断，如果返回的时候发现曾经发生过中断，则在退出acquire()之前，调用一下selfInterrupt自我中断一下，就好像将这个发生在枪锁过程中的中断"推迟"抢锁结束以后一样。  
  
  从上面的介绍可以看到，除了获取锁的逻辑tryAcquire(arg)由子类实现一样，其余方法均由AQS实现。  

在JDK1.8中，公平锁的逻辑在tryAcquire方法中实现，非公平锁的逻辑在lock方法和tryAcquire方法中都有实现。  

在JDK11 中，公平锁和非公平锁的逻辑均在tryAcquire方法中实现。  

### 公平锁  

```java
public void lock() {
    sync.lock();
}

/**
* 公平锁的lock方法在ReentrantLock的FairLock内部类中实现，调用了AbstractQueuedSynchronizer中的acquire()
*/
final void lock() {
    acquire(1);
}

/**
* AbstractQueuedSynchronizer中的acquire方法，获取锁的流程
*/
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

/**
* 实现公平锁的逻辑
*/
protected final boolean tryAcquire(int acquires) {
    //记录一下当前的线程
    final Thread current = Thread.currentThread();
    //获取当前锁的状态
    int c = getState();
    /**
    * c=0，说明当前锁是avaiable的，没有任何线程占有可以尝试获取锁
    * 因为是公平锁，所以在抢占锁之前会查看队列中有没有排在当自己之前的Node，
    * 如果有，意味着在队列中有线程在排队，为了符合公平锁的逻辑，该线程也需要
    * 进入到队列中等待锁
    * 如果没有，说明没有线程在排队，那么该线程可以通过cas操作的方式获取锁，然
    * 后退出
    */
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
            /*为了阅读方便, hasQueuedPredecessors源码就直接贴在这里了, 
            这个方法的本质实际上是检测自己是不是head节点的后继节点，即处在
            阻塞队列第一位的节点
            public final boolean hasQueuedPredecessors() {
                Node t = tail; 
                Node h = head;
                Node s;
                return h != t && ((s = h.next) == null || s.thread !=Thread.currentThread());
            }
            */
            //通过cas的方式设置 state为1，acquires为tryAcquire传入的参数，是1
            compareAndSetState(0, acquires)) {
            //设置当前线程为占有锁的线程
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    /*
    * 如果c>0，说明锁已经被占用，但是还不会退出
    * 这是因为可重入锁，还需要检测占用锁的线程是不是当前线程。如果是当前线程，说明已经拿到了锁，可以直接重入
    * 这是需要将c+1
    * 这里没有设置c的状态没有使用cas操作，是因为当前线程已经拿到了锁，不存在竞争的问题，所以就不要进行cas操作了
    */
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    //这里说明已经有线程拿到了锁，并且占用锁的线程还不是当前线程，所以返回false
    return false;
}
```

### 非公平锁  

```java
public void lock() {
    sync.lock();
}

/**
* 非公平锁的lock方法在ReentrantLock的NonFairLock内部类中实现，可以看到lock方法首,线程先以cas操作的方式尝试
* 设置state的值为1，如果设置成功了，就说明了该线程获取锁成功，接下来只需要设置当前线程持有了锁即可；如果设置失败
* 说明了锁已经被占用了，接下来就去执行AbstractQueuedSynchronizer的acquire(1)这个方法
* 这个方法这也体现了非公平锁的逻辑
*/
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}

/**
* AbstractQueuedSynchronizer中的acquire方法，获取锁的流程
*/
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

protected final boolean tryAcquire(int acquires) {
    //非公平锁的内部又调用了nonfairTryAcquire(acquires)方法
    return nonfairTryAcquire(acquires);
}

final boolean nonfairTryAcquire(int acquires) {
    //记录一下当前线程
    final Thread current = Thread.currentThread();
    //获取当前锁的状态
    int c = getState();
    /*
    * c=0，说明当前锁是avaiable的，没有任何线程占有，所以可以尝试获取锁
    * 因为是非公平锁，所以在抢占锁之前不像公平锁那样查看队列中是否还有排队的线程
    * 而是在发现如果没有线程占有锁的时候，就会直接尝试自己去占有锁，这里就体现了
    * 非公平锁的逻辑
    */
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            //设置当前线程持有了锁
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    /*
    * 如果c>0，说明锁已经被占用，但是还不会退出
    * 这是因为可重入锁，还需要检测占用锁的线程是不是当前线程。如果是当前线程，说明已经拿到了锁，可以直接重入
    * 这是需要将c+1
    * 这里没有设置c的状态没有使用cas操作，是因为当前线程已经拿到了锁，不存在竞争的问题，所以就不要进行cas操作了
    */
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    //这里说明已经有线程拿到了锁，并且占用锁的线程还不是当前线程，所以返回false
    return false;
}

```

由上面可以看出，获取锁其实就是在更改state的状态，将state的状态由0更新到1。公平锁和非公平锁的区别也就在于，公平锁在更改state的状态之前，会先查看队列中是否存在正在排队的线程。如果没有正在排队的线程，才会去尝试更新state的状态，即获取锁。非公平锁，则会不会查看队列中是否存在正在排队的线程，而是直接尝试更新state的状态，这也正体现了非公平锁的含义。  

而进入到重入锁的操作时，不再使用cas操作，则是因为当前线程已经获取到了锁，就不存在竞争的问题，也就不需要使用cas操作了。  

无论公平锁还是非公平锁，tryAcquire以后的逻辑都是一样的。 

### addWaiter方法  

无论是公平锁还是非公平锁，执行到addWaiter方法，说明前面尝试获取锁的tryAcquire方法失败了，既然执行失败了，那么就需要将当前线程包装成一个Node节点，加入到等待队列中去。加入等待队列的逻辑是有addWaiter来实现的。  

```java
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    /**
    *  static finAl Node EXCLUSIVE = null;
       Node(Thread thread, Node mode) {     // Used by addWaiter
            this.nextWaiter = mode;
            this.thread = thread;
        }
        Node的nextWaiter被设置为Node.EXCLUSIVE,即为空
    */
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    //如果队列不为空，则使用cas操作的方式将当前节点设置为尾节点
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    /**
    * 到这里来有两种可能
	* 1. 队列为空，
	* 2. cas操作失败
    */
    enq(node);//将节点插入到队列中
    return node;
}
```

  插入尾节点如下图所示

![添加尾节点](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E6%B7%BB%E5%8A%A0%E5%B0%BE%E8%8A%82%E7%82%B9.png)  

可见在每一个处于独占锁模式下的节点，他的nextWaiter一定是null。  

在这个方法中，首先会尝试直接从队尾入队。但是在并发条件下同一时刻存在多个线程尝试入队的可能性，导致compareAndSetTail(pred, node)失败（这是因为已经有其他线程设置了尾节点，导致尾节点不再是之前perd所指向的节点了）。

如果入队失败或者队列本身就是一个空队列，这时就需要调用enq(node)，该方法会通过自旋+cas操作的方式，确保节点加入当前队列。  

### enq方法  

执行到这里的原因可能是以下两种之一  

1. 等待队列是空的，没有线程等待  

2. 其他线程在当前线程入队的过程中率先完成了入队，导致尾节点发生了改变，cas操作失败  

在该方法中，以自旋（死循环）的方式将节点插入队列，如果失败则不停的尝试，直到成功为止。而且此方法也在队列为空时，初始化队列。这也说明，队列是延迟初始化的  

```java
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        /**
        * 如果是空队列，首先进行初始化
        * 这里也可以看出，队列不是在构造的时候初始化的，而是延迟到需要用的时候再初始化，以提升性能
        */
        if (t == null) { // Must initialize
            //这里，将head节点设置为一个空节点
            if (compareAndSetHead(new Node());
                /**
                * 空队列的时候，第一次循环将head节点设置为一个空节点
                * 第二次循环的时候才会走到下面的else逻辑中
                * 就是将其余的节点和第一个节点的添加的逻辑做成一样的
                */
                tail = head;
        } else {
            /**
            * 到这里说明不是空队列了，不断尝试从尾部将节点加入队列
            */
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

这里需要注意的是，当队列为空时，初始化队列并没有使用当前传进来的节点而是新创建了一个空节点。当新建完空的头节点之后，并没有立即返回，而是在下一次循环中，将包装了当前线程的Node节点加入到这个空节点的后面。这就意味着，队列的头节点不代表任何等待的线程。  

### 尾分叉  

```java
else {
    /**
    * 到这里说明不是空队列了，不断尝试从尾部将节点加入队列
    */
    node.prev = t;
    if (compareAndSetTail(t, node)) {
        t.next = node;
        return t;
    }
```

将Node节点加入到同步队列的末尾需要三步：  

1. 设置node的前去节点为当前的尾节点：node.prev = t

2. 修改tail属性，使它指向当前节点

3. 修改原来的尾节点，使它的next指针指向当前节点

![enq方法](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/enq%E6%96%B9%E6%B3%95.png)

但需要注意的是，这里的三步不是一个原子操作，第一步很容易成功；而第二步是一个cas操作，在并发条件下有可能会失败，第三步只有在第二部成功的情况下才会执行。这里的cas操作保证了同一时刻只有一个节点能够称为尾节点，其他的节点将失败，然后回到for循环中继续重试。  

所以当有大量的线程同时入队的时候，同一时刻，只有一个线程能够完成这三步，其余的线程完成第一步，于是就出现了尾分叉  

![尾分叉](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E5%B0%BE%E5%88%86%E5%8F%89.png)  

这里第三步是在第二步成功之后才执行的，这就意味着，有可能已经完成了第二步，将新节点设置成了尾节点，此时原来旧的尾节点的next属性值为null（因为还没有来得及执行第三步）。如果此时恰好有线程从头到尾的遍历队列，则它遍历不到新加进来的尾节点，这显然是不合理的。  

另一方面，当完成第二步时，第一步一定是完成的，也就是节点的prev属性指向的是它的前驱节点。此时如果有线程从当前节点开始向前遍历，肯定可以遍历到所有的节点。因此在AQS的源码中，经常可以看到从尾节点开始遍历而不是从头节点开始遍历。因为一个节点可以加入到队列，说明它的prev属性一定是指向它的前驱节点，但是next属性值有可能为null。  

### addWaiter总结  

addWaiter(Node.EXCLUSIVE)方法并不涉及到任何关于锁的操作，就是解决了并发条件下的节点入队问题。具体来说就是该方法保证了将当前线程包装成立一个Node节点加入到等待队列的队尾。如果队列为空，则会创建一个新的空节点作为头节点，再将当前节点作为头节点的后继节点。 

 addWaiter方法最终返回了代表当前线程的Node节点，在返回的那一刻，这个节点必然是当时队列的尾节点。  

再回到获取锁的逻辑中  

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

当执行完addWaiter方法后，节点现在被添加到等待队列中，接下来将执行acquireQueued方法。  

### acquireQueued方法  

首先有几点要说明：  

1. 执行到这里，说明addWaiter方法已经成功地将包装了当前Thread线程的节点添加到了等待队列的结尾  

2. 该方法中再次尝试获取锁  

3. 再尝试再次获取锁失败后，判断是否需要把当前线程挂起  

为什么前面获取锁失败了，这里还要再次尝试获取锁呢？因为存这样的可能就是当前节点的前驱节点就是HEAD节点。因为HEAD节点是个空节点，不代表任何线程。如果当前节点的前驱节点是HEAD节点，说明当前节点已经是排在整个等待队列的最前面了。  

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

这里又有一个自旋操作for(;;)。来一段一段看  

```java
final Node p = node.predecessor();
if (p == head && tryAcquire(arg)) {
    setHead(node);
    p.next = null; // help GC
    failed = false;
    return interrupted;
}
```

首先获取尾节点的前驱节点（因为上一步返回的就是尾节点，并且这个节点就是代表了当前线程的Node）。

如果前驱节点是head节点，说明当前节点已经是排在了队列的最前面，所以这里再次尝试获取锁。如果这一次获取锁成功，即tryAcquire方法返回true，即进入if代码块，调用setHead方法  

```java
private void setHead(Node node) {
    head = node;
    node.thread = null;
    node.prev = null;
}
```

这个方法将head指针指向当前node节点，并且将节点的thread、prev属性置为null  

![setHead](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/setHead.png)  

可以看出，这个方法就是将head指针指向已经获得了锁的节点。但是又将node的thread和prev属性设置为了null，某种意义上来说，新的头节点又被设置成了一个新的空节点，不代表任何线程。这是因为tryAcquire调用成功后，exclusiveOwnerThread属性就已经被设置成了当前获取锁的线程，此处就没有必要再记录了。其实这就相当于是一个出队的操作。

这里setHead方法没有采用cas的操作，是因为该线程已经获取到了锁，就不会产生竞争了。  

接下来看另一种情况，即p == head && tryAcquire(arg)返回false，此时需要判断是否需要将当前线程挂起。  

### shouldParkAfterFailedAcquire 方法  

从函数名也可以看出，该方法用于决定在获取锁失败后，是否将线程挂起。而这个决定的依据就是前驱节点的withStatus属性。  

再来看一下withStatus的属性  

```java
volatile int waitStatus;
static final int CANCELLED =  1;
static final int SIGNAL    = -1;
static final int CONDITION = -2;
static final int PROPAGATE = -3;
```

一共有四种属性，在独占锁中用到了其中的两种**CANCELLED**和**SIGNAL**。

**CANCELLED**：状态表示Node所代表的线程已经取消了排队，即放弃了锁的获取。  

**SIGNAL**：这个状态不是表示当前节点的状态，而是表示其后继节点的状态。当一个节点的withStatus被设置成了SIGNAL，那么就说明它的后继节点已经被挂起或者马上要被挂起，因此在当前节点释放了锁或者放弃了锁的时候，它还需要有一个额外的操作，来唤醒它的后继节点。  

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;//获取当前节点的前驱节点的withStatus
    if (ws == Node.SIGNAL)//前驱节点的withStatus的状态被设置成了SIGNAL,那么当前节点可以放心的被挂起
        /**
        * This node has already set status asking a release
        * to signal it, so it can safely park.
        */
        return true;
    if (ws > 0) {
        /**
        * 如果前驱节点的withStatus>0，说明状态为CANCELLED，即前驱节点已经放弃了锁的获取
        * 既然前驱节点不等待锁了，那么就继续向前寻找，直到找到一个还在等待锁的节点
        * 然后跨过这些不等待锁的节点，直接排在等待锁的节点的后面
        * Predecessor was cancelled. Skip over predecessors and
        * indicate retry.
        */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        /**
        * 如果前驱节点的状态既不是CANCELLED，又不是SIGNAL，就用cas的操作给前驱节点设置一个SNGNAL的状态
        * waitStatus must be 0 or PROPAGATE.  Indicate that we
        * need a signal, but don't park yet.  Caller will need to
        * retry to make sure it cannot acquire before parking.
        */
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

可以看出shouldParkAfterFailedAcquire就做了以下几件事： 

+ 如果前驱节点的withStatus的值是Node.SIGNAL，那么直接返回true；
+ 如果前驱节点的withStatus的值是Node.CANCELLED，那么继续向前寻找，直到找到等待中的前驱节点，然后当前节点排在它后面，返回false；
+ 其他情况，将前驱节点的状态设置为Node.SIGNAL，然后返回false；

这个方法只有在前驱节点的withStatus的值是Node.SIGNAL时才会返回true，其他情况都会返回false。在回到这个方法的调用处：  

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
           //在这里调用
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

可以看到，如果shouldParkAfterFailedAcquire返回false，会自己回到循环中再次尝试获取锁，这是因为当前节点的前驱节点可能变成了head节点；如果shouldParkAfterFailedAcquire返回true，在可以安心的将当前线程挂起，此时将调用  parkAndCheckInterrupt方法。  

### parkAndCheckInterrupt方法  

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);//线程执行到这里，将会被挂起，不再向下执行
    return Thread.interrupted();
}
```

## 锁的获取的总结  

+ 在AQS中使用state属性表示锁，如果能够成功将state属性通过cas操作从0设置为1，即获取成功了锁  

+ 获取了锁的线程才能将exclusiveOwnerThread设置成当前获取锁的线程  

+ addWaiter方法负责将当前等待锁的线程包装成Node，并成功添加到等待队列的队尾，这一点是由enq方法来保证的，而且enq方法还负责等待队列的初始化工作，就是新建一个空的头节点  

+ acquireQueued方法用于在Node成功入队以后，尝试继续获取锁（取决于前驱节点是不是head），或者将当前线程挂起  

+ shouldParkAfterFailedAcquire方法用于保证当前节点的前驱节点的withStatus属性值为Node.SIGNAL，从而保证当前线程可以被安全挂起，前驱节点会负责将当前节点唤醒  

+ parkAndCheckInterrupt会挂起当前线程，并检查中断状态  

+ 如果最终成功获取了锁，线程会从lock方法返回，继续向下执行；如果失败了，线程则会阻塞。  

## 锁的释放

由于锁的释放操作对于公平锁和非公平锁都是一样的，所以unlock的逻辑并没有放在FairSync或者NonFairSync中，而是直接定义在ReentrantLock中  

```java
/**
* 该方法在ReentrantLock类中实现，因为公平锁和非公平锁释放锁的逻辑完成相同
*/
public void unlock() {
        sync.release(1);
}

/**
* 该方法定义在AQS中
*/
public final boolean release(int arg) {
    if (tryRelease(arg)) {//tryRelease方法由AQS的子类自己实现，是释放锁的逻辑实现
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);//醒后继节点锁代表的线程
        return true;
    }
    return false;
}
```

unlock方法调用了AQS类的release方法  

### release方法  

这个方法描述了锁的释放流程   

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {//tryRelease方法由AQS的子类自己实现，是释放锁的逻辑实现
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);//醒后继节点所代表的线程
        return true;
    }
    return false;
}
```

### tryRelease方法  

tryRelease方法由ReenrantLock的内部静态类Sync来实现，释放锁的前提是已经获取了锁，所以在释放锁的时候不需要再进行cas操作了，因为不会产生竞争问题  

```java
protected final boolean tryRelease(int releases) {
    //首先将当前锁的线程个数减一，因为传入的releases为1
    //这里c可能会大于0，这主要是针对重入锁的情况
    int c = getState() - releases;
    //释放锁的线程当前必须是锁的持有线程
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    //如果c=0，说明没有线程持有锁，也就是锁已经被完全释放了
    if (c == 0) {
        free = true;
        //设置exclusiveOwnerThread为null，即持有锁的线程为null
        setExclusiveOwnerThread(null);
    }
    //设置state的值为0
    setState(c);
    return free;
}
```

  释放锁成功之后会执行接下来的逻辑  

```java
Node h = head;
//h==null 或者h.withStatus==0，说明队列中没有需要被唤醒的进程，既然也就不需要唤醒后继节点
if (h != null && h.waitStatus != 0)
    unparkSuccessor(h);//醒后继节点所代表的线程
return true;
```

这里h !=null很好理解，如果为空，说明队列为空，当然不需要唤醒后继节点

那么h.withStatus!=0呢？之前说过，后继节点挂起的前提是其前驱节点的withStatus=Node.SIGNAL，即withStatus=-1；但是还有另外一处设置withStatus值的地方，就是新建队列的时候，新建一个空的头节点，此时头节点的withStatus会被设置为0。所以当一个head节点的withStatus为0说明了此时等待队列中除了头节点没有其他需要被唤醒的节点，因为如果有的话，后继节点在加入队列的时候会在shouldParkAfterFailedAcquire方法中将其前驱节点的withStatus设置为-1  

```java
else {
        /**
        * 如果前驱节点的状态既不是CANCELLED，又不是SIGNAL，就用cas的操作给前驱节点设置一个SNGNAL的状态
        * waitStatus must be 0 or PROPAGATE.  Indicate that we
        * need a signal, but don't park yet.  Caller will need to
        * retry to make sure it cannot acquire before parking.
        */
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
}
```

接下来看看unparkSuccessor  

### unparkSuccessor方法  

```java
private void unparkSuccessor(Node node) {
    /**
    * 如果当前节点的withStatus比0小，将它设置为0，类似于一个空的头节点
    * If status is negative (i.e., possibly needing signal) try
    * to clear in anticipation of signalling.  It is OK if this
    * fails or if status is changed by waiting thread.
    */
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    /**
    * 通常情况下，要唤醒的节点就是自己的后继节点
    * 如果后继节点存在且也在等待锁，那么就直接唤醒它
    * 但是有可能存在后继节点取消等待锁的情况
    * 此时需要从等待队列的队尾开始向前遍历，直到找到距离head节点最近的且withStatus<0的节点
    * Thread to unpark is held in successor, which is normally
    * just the next node.  But if cancelled or apparently null,
    * traverse backwards from tail to find the actual
    * non-cancelled successor.
    */
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    //如果找到了还在等待锁的节点，则唤醒它
    if (s != null)
        LockSupport.unpark(s.thread);
}

```

另外一个问题就是为什么要从尾节点开始逆向遍历查找，而不是直接从head节点开始向后查找。

首先需要知道，从后往前查找是基于一定的条件的，即后继节点不存在或者后继节点取下了排队。

```java
if (s == null || s.waitStatus > 0)
```

从后往前遍历是因为之前提到的尾分叉的问题，因为入队操作不是一个原子操作。可能存在一个节点的next还没有来得及指向后继的节点，而加入队列的操作中，节点的prev肯定是被指向了其前驱的节点。所以从后向前遍历一定是最精确的。  

最后调用了LockSupport.unpark(s.thread)，唤醒了后继线程，接下来会发生什么呢？

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);//因为在这里被挂起，所以线程唤醒后，继续从这里运行
    return Thread.interrupted();
}
```

调用Thread.interrupted方法，这个方法会返回当前线程（之前被挂起的这个线程）的中断状态，并清除。接着再返回到parkAndCheckInterrupt被调用的地方。  

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 我们在这里！在这里！！在这里！！！
            // 我们在这里！在这里！！在这里！！！
            // 我们在这里！在这里！！在这里！！！
            if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

可见，如果Thread.interrupted返回true，则parkAndCheckInterrupt就会返回true，if条件成立，interrupted被设置为true;如果Thread.interrupted返回false，则interrupted仍然为false。接下来就会回到for(;;)死循环的开头，进行新一轮的抢锁。  

假设这次抢到了锁，将从return interrupted处返回。返回到acquireQueued的调用处  

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

可以看到如果acquireQueued方法返回false，将会执行selfInterrupt方法  

```java
static void selfInterrupt() {
    Thread.currentThread().interrupt();
}
```

而它的作用就是中断当前线程。

为什么要中断当前线程呢？这是因为我们不知道当前线程被唤醒的原因是什么？  

当前线程被唤醒可能是因为有线程释放了锁，执行了LockSupport.unpark方法，也有可能是因为当前线程再等待中被中断了，因此通过Thread.interrupted方法来记录当期线程的中断状态。以便再最后返回acquire之后，发现当前线程曾经被中断过，那么就需要再中断一次。  

从上面的代码可以知道，即使线程在等待资源的过程中被中断唤醒，它还是会不依不饶的再抢锁，直到它抢到锁为止。也就是说它不会响应这个中断，仅仅是记录下自己曾经被中断过。最后，当抢到锁返回了，发现自己曾经被中断过，就再中断一次，将这个中断弥补上。  

注意：中断对一个线程来说只是一个建议，一个线程的中断状态被设置为了true，线程可以选择忽略这个中断，中断一个线程并不影响线程的执行。  

## 锁的释放的总结  

+ 首先需要将锁state的值减去1，如果为0，说明锁被彻底释放；如果>0，则对应可重入锁的情况  

+ 彻底释放锁之后，将exclusiveOwnerThread设置为null，表示当前锁的持有线程为空，然后再等待队列中唤醒后继等待锁的节点

+ 一般情况下，头节点的后继节点再等待锁，所以需要将它唤醒；也存在后继节点为空或者后继节点取消等待的可能，这时就需要从队尾向队头遍历，找到距离头节点最近并且withStatus<0的节点，将这个节点所代表的线程唤醒  

+ 唤醒后，需要记录一下该线程的中断状态；如果中断状态为true，则在执行完acquireQueue方法后，中断一下当前线程，将之前的中断弥补上  



