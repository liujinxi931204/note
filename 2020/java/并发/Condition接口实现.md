## Condition接口  

Lock接口中定义了newCondition方法，返回一个关联在Lock对象上的Condition对象。  

Condition接口的出现是为了扩展Synchronized中的wait、notify的机制。那么wait、notify有什么弊端呢？我们知道调用了wait的线程，都会被放在ObjectMonitor中的wait set中。这样一来就有一个问题就是所有的调用了wait的线程都会被放到ObjectMonitor的wait set中。这样就有可能会被唤醒后，抢到了锁，但是发现条件还不满足，这时还得调用wait挂起，从而导致了无意义的时间和CPU资源的浪费。产生这一切的根源在于，调用wait方法时没有办法来指明究竟在等待什么条件。解决的办法就是在挂起时指明在等待什么条件，同时，在等待事件发生后，只唤醒等待在这个事件或者条件上的线程，condition接口的实现就是这个思路。  

## 实现 

通过wait/notify机制来对比await/signal机制  

+ 调用wait方法的线程首先必须是已经进入了同步代码块，即已经获取了监视器锁；与之类似，调用await方法的线程首先必须获得lock锁。  
+ 调用wait方法会释放已经获得的监视器锁，进入当前监视器锁的等待队列（wait set）中；与之对应，调用await方法的线程会释放已经获得的lock锁，进入到当前Condition对应的条件队列中。
+ 调用监视器锁的notify方法会唤醒等待在该监视器上的线程，这些线程开始参与锁的竞争，并在获得锁后，从wait方法处恢复执行；与之类似，调用Condition的signal方法会唤醒对应的条件队列中的线程，这些线程将开始参与锁竞争，并在获得锁后，从await方法处恢复执行。

## 实战  

```java
class BoundedBuffer{
    final Lock lock = new ReentrantLock();
    final Condition notFull = lock.newCondition();
    final Condition notEmpty = lock.newCondition();
    final Object[] items = new Object[100];
    int putptr,takeptr,count;
    
    //生产者方法，向数组中写入数据
    public void put(Object x) throws InterruptedException{
        lock.lock();
        try{
            while(count == items.length){
                //数组已满，没有空间时，挂起等待，直到数组有空间了
                notFull.await();
            }
            itmes[putptr]=x;
            if(++putptr==items.length) putptr=0;
            ++count;
            //因为放入了一个数据，数组肯定不为空了，此时唤醒等待notEmpty条件上的线程
            notEmpty.signal();
        }finally{
            lock.unlock();
        }
    }
    
    public void take() throws InterruptException{
        lock.lock();
        try{
            while(0==items.length){
                //数组为空，没有数据可以获取，挂起等待
                notEmpty.await();
            }
            Object x=items[takeptr];
            if(++takeptr==items.length) takeptr=0;
            --count;
            notFull.signal();
        }finally{
            lock.unlock();
        }
    }
}
```

这是一个典型的生产者-消费者模型。这里在同一个lock锁上，创建了两个条件队列notFull和notEmpty。当数组满时，put方法在notFull条件上等待，直到数组有空间；当数组空时，take方法在notEmpty条件上等待，直到数组中有数据。而notFull.signal和notEmpty.signal则是用来唤醒在这个条件上等待的线程。  

注意：上面在条件队列上等待，被唤醒后线程响应的线程会进入到之前说的等待队列中去争抢锁。争抢到锁的线程才能在await返回，继续执行。因此这里牵扯到两种队列，condition queue和sync queue，都定义在AQS中。  

## 同步队列VS条件队列  

### sync queue  

![CLH队列](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/CLH%E9%98%9F%E5%88%97.png)

sync queue是一个双向链表，使用prev、next属性来串联节点。但是在同步队列中，始终没有用到nextWaiter属性，即使是在共享模式下，这一属性也仅仅是作为一个标志，指向了一个空节点。因此，在sync queue中，不会使用nextWaiter属性来串联节点。  

### condition queue

![preview](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/1460000016462284)  

每一个Condition对象对应一个Condition队列，每个Condition队列都是独立的，互相不影响的。在上图中，如果调用notFull.await()方法，则当前线程会被包装成Node节点然后加入到notFull队列的末尾。  

值得注意的是，condition queue是一个单向链表，在该链表中使用nextWaiter属性来串联链表。就像在sync queue队列中不会使用nextWaiter来串联节点一样，在condition queue中不会使用prev、next来串联节点，它们的值都为null。也就是说，在condition queue中，真正使用到的属性只有三个  

+ thread：表示当前正在等待某个条件的节点  

+ waitStatus：条件等待的状态  

+ nextWaiter：指向条件队列中下一个节点  

在条件队列中，只需要关注waitStatus的值是不是CONDITION就好了。如果waitStatus的值是CONDITION，说明线程处于正常的等待状态；如果不是，说明该线程不再等待，此时需要从条件队列中出队。  

### sync queue和condition queue的联系  

![preview](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/1460000016462285)  



一般情况下，等待锁的sync queue和条件队列condition queue是相互独立的，彼此之间没有任何联系。但是，当调用某个条件队列的signal方法时，会唤醒某个或者所有在这个条件队列中的线程，被唤醒的线程和普通线程一样去争抢锁，如果没有争到，同样要被加入到sync queue中，此时节点就从condition queue中被转移到sync queue  

有两点需要注意：  

1. 条件队列中的节点是一个一个的转移到等待队列中，哪怕调用的是signalAll方法，也是一个一个转移过去的，而不是将整个队列连接在等待队列最后  

2. 在condition queue中使用nextWaiter来串联链表，没有使用prev、next属性；在sync queue中使用prev、next来串联链表而没有使用nextWaiter属性。所以在转移的过程中，需要断开原有的nextWaiter，然后连接prev、next，这在某种程度上也是一个一个转移过去的原因  

## 入队和出队时锁的状态  

sync queue是等待锁的队列，当一个线程被包装成Node加入到该队列时，必然是没有获取到锁的；当获取到了锁之后，必然会从该队列中移除。  

condition队列是等待在特定条件下的队列，因为调用await方法时，必然是已经获得了锁，所以在进入condition队列前线程必然是已经获取到了锁；在被包装成Node节点加入到condition队列后，线程将释放锁，然后挂起；当处于该队列中的线程被signal方法唤醒后，由于队列中的节点在之前挂起时已经释放了锁，所以必须先再次争抢锁，因此，该节点会被添加到sync队列中。因此条件队列在出队时，线程并不持有锁。  

+ condition queue：入队时已经持有了锁—>在队列中释放锁—>离开队列时没有锁—>转移到sync queue  

+ sync queue：入队时没有锁—>在队列中竞争锁—出队时持有了锁  

## ConditionObject  

AQS对Condition这个接口的实现主要是通过ConditionObject，它的核心就是一个条件队列，每一个在某个condition上等待的线程都会被封装成Node对象加入到这个队列中。  

### 核心属性  

它的核心属性只有两个：  

```java

/** First node of condition queue. */
private transient Node firstWaiter;
/** Last node of condition queue. */
private transient Node lastWaiter;
```

这两个属性分别代表了条件队列的队头和队尾，每新建一个conditionObject对象，都会对应一个条件队列  

### 构造函数  

```java
public ConditionObject() { 
}
```

可见条件队列应该是延时初始化的，在真正用到的时候才会初始化  

### 接口方法  

#### await方法  

```java
public final void await() throws InterruptedException {
    //如果当前线程在调用await方法之前已经被中断了，则直接抛出异常InterruptedException
    if (Thread.interrupted())
        throw new InterruptedException();
    //将当前线程包装成Node节点，加入到条件队列中
    Node node = addConditionWaiter();
    //释放当前线程锁占用的锁，保存当前的锁状态
    int savedState = fullyRelease(node);
    int interruptMode = 0;
    //如果当前节点不在同步队列中，说明刚刚被await，还没有线程调用signal方法，则直接将当前线程挂起
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);//当前线程在这里被挂起，后面被唤醒以后，同样从这里开始执行
        //线程执行到这里说明要么是被signal方法唤醒，要么是线程被中断
        //所以下面检查中断状态，如果是被中断，则跳出while循环
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
```

#### addConditionWaiter方法  

这个方法见名知义，就是将当前执行了await方法的线程包装成一个Node节点，然后加入到条件队列中。  

```java
private Node addConditionWaiter() {
    Node t = lastWaiter;
    // If lastWaiter is cancelled, clean out.
    //如果尾节点的waitStatus不是CONDITION，从前面可以知道，需要在条件队列中进行出队操作
    //unlinkCancelledWaiters方法会从头到尾遍历整个队列，清理掉所有不是CONDITION的节点
    if (t != null && t.waitStatus != Node.CONDITION) {
        unlinkCancelledWaiters();
        t = lastWaiter;
    }
    /**
    * 创建一个Node节点，保存当前线程以及设置waitStaus为Node.CONDITION
    * 如果lastWaiter为null，说明整个条件队列为空，这是先让firstWaiter指向当前节点
    * 如果lastWaiter不为null，说明条件队列不为空，就将当前节点的nextWaiter指向当前节点
    * 最后尾节点指向当前节点
    */
    Node node = new Node(Thread.currentThread(), Node.CONDITION);
    if (t == null)
        firstWaiter = node;
    else
        t.nextWaiter = node;
    lastWaiter = node;
    return node;
}
```

这里没有使用cas的原因是调用await方法的前提是当前线程已经拿到锁了，所以就不存在锁竞争的问题了，即不存在并发问题。

注意这里和加入等待队列有所不同  

+ 节点加入sync queue时，waitStatus的值为0，但节点加入condition queue时，waitStatus被设置为Node.CONDITION  
+ 等待队列的头节点是一个空节点，如果队列为空，会首先创建一个空的头节点，然后进行入对的操作；条件队列的firstWaiter和lastWaiter不是空节点，分别指向队列的第一个节点和最后一个节点  
+ sync queue队列是一个双向队列，在加入节点后需要修改当前节点的前驱节点和后继节点；而conditon queue是一个单向队列，使用nextWaiter来连接后继节点

如果发现尾节点不是CONDITIO的状态，那么新加入的节点就不应该排在其后，这时就需要调用unlinkCancelledWaiters方法来清理所有不是CONDITION状态的节点  

```java
private void unlinkCancelledWaiters() {
    Node t = firstWaiter;
    Node trail = null;
    while (t != null) {
        Node next = t.nextWaiter;
        if (t.waitStatus != Node.CONDITION) {
            t.nextWaiter = null;
            if (trail == null)
                firstWaiter = next;
            else
                trail.nextWaiter = next;
            if (next == null)
                lastWaiter = trail;
        }
        else
            trail = t;
        t = next;
    }
}
```

该方法从头开始遍历，这里使用t和trail来记录队列的firstWaiter和最后一个waitStatus为CONDITION的节点。  

#### fullyRelease方法  

当前节点加入到条件队列后，接下来就需要释放当前线程所持有的锁了。这个方法就是fullyRelease方法  

```java
final int fullyRelease(Node node) {
    boolean failed = true;
    try {
        //获取当前AQS的state的值，记录当前的state值，主要针对锁重入的情况
        //针对可重入锁，一次性全部释放
        int savedState = getState();
        if (release(savedState)) {
            failed = false;
            return savedState;
        } else {
            /**
            * 这里有可能抛出异常的原因在于fullRelease方法没有判断当前锁的持有者是不是当前线程
            * 如果不是当前线程，release方法会返回false，首先进入到finally中，将当前节点的
            * waitStatus设置为CANCELLED，然后抛出异常
            * finally中执行的优先于try/catch中的return throw
            */
            throw new IllegalMonitorStateException();
        }
    } finally {
        if (failed)
            node.waitStatus = Node.CANCELLED;
    }
}
```

由上面的分析可以知道，对于可重入锁这里是一次性全部释放的。  

当发现当前线程不是持有锁的线程，release就会返回false。此时failed依然还是true，进入到finally中将当前节点的waitStatus设置为Node.CANCELLED，然后进入else中抛出IllegalMonitorStateException异常。这也就是为什么上面在入队的时候先检查尾节点是不是已经被取消了。  

在当前线程的锁被完全释放了以后，就可以调用LockSupport.part(this)方法将当前线程挂起，等待以后signal或者signalAll来唤醒。但是，在挂起之前，先用isOnSyncQueue确保当前节点不在sync queue中，这是为什么呢？当前线程不是在条件队列中吗？又怎么会出现在sync queue的情况呢？  

```java
final boolean isOnSyncQueue(Node node) {
    if (node.waitStatus == Node.CONDITION || node.prev == null)
        return false;
    //在条件队列中，prev、next都会为null，使用nextWaiter来连接后继节点
    if (node.next != null) // If has successor, it must be on queue
        return true;
    /*
         * node.prev can be non-null, but not yet on queue because
         * the CAS to place it on queue can fail. So we have to
         * traverse from tail to make sure it actually made it.  It
         * will always be near the tail in calls to this method, and
         * unless the CAS failed (which is unlikely), it will be
         * there, so we hardly ever traverse much.
         */
    return findNodeFromTail(node);
}
```

```java
/**
     * Returns true if node is on sync queue by searching backwards from tail.
     * Called only when needed by isOnSyncQueue.
     * @return true if present
     */
private boolean findNodeFromTail(Node node) {
    Node t = tail;
    for (;;) {
        if (t == node)
            return true;
        if (t == null)
            return false;
        t = t.prev;
    }
}
```

为了解释这一问题，首先来看一下signal方法  

#### signalAll方法  

在看signalAll方法之前，先要区分调用signalAll方法的线程和signalAll方法要唤醒的线程（在条件队列中的线程）  

+ 调用signalAll方法的线程一定是已经获取到了锁的线程，现在准备释放锁了  

+ 在条件队列中的线程已经在对应的条件上挂起了，等待着被唤醒，然后去争抢锁  

```java

/**
         * Moves all threads from the wait queue for this condition to
         * the wait queue for the owning lock.
         *
         * @throws IllegalMonitorStateException if {@link #isHeldExclusively}
         *         returns {@code false}
         */
public final void signalAll() {
    //首先检查调用signalAll方法的线程是不是当前持有锁的线程，isHeldExclusively由AQS的子类来实现
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    //如果条件队列不为空
    if (first != null)
        //唤醒条件队列中的所有节点
        doSignalAll(first);
}
```

首先调用各个AQS子类实现的isHeldExclusively方法来判断当前调用signalAll方法的线程是不是当前线程。如果是的话，通过firstWaiter判断条件队列是否为空，如果条件队列不为空，则调用doSignalAll方法来唤醒队列中的所有节点。  

```java
private void doSignalAll(Node first) {
    //清空整个队列
    lastWaiter = firstWaiter = null;
    do {
        Node next = first.nextWaiter;
        first.nextWaiter = null;
        //添加到sync queue队列的末尾
        transferForSignal(first);
        first = next;
    } while (first != null);
}
```

首先通过lastWaiter=firstWaiter=null来清空整个队列，然后通过do-while循环将原先队列里的节点一个一个取出来，通过transferForSignal方法添加到sync queue队列的末尾。  

```java
final boolean transferForSignal(Node node) {
    /*
    * If cannot change waitStatus, the node has been cancelled.
    * 设置节点的waitStatus失败，说明当前节点在signalAll之前已经被取消了，则跳过这个节点
    */
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;

    /*
    * Splice onto queue and try to set waitStatus of predecessor to
    * indicate that thread is (probably) waiting. If cancelled or
    * attempt to set waitStatus fails, wake up to resync (in which
    * case the waitStatus can be transiently and harmlessly wrong).
    * 通过enq方法将当前节点添加到sync queue队列的末尾，注意enq方法返回的是当前节点的前驱
    * 如果前驱节点的状态是取消状态或者无法设置前驱节点的状态为SIGNAL，就唤醒当前的线程
    */
    Node p = enq(node);
    int ws = p.waitStatus;
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        LockSupport.unpark(node.thread);
    return true;
}
```

