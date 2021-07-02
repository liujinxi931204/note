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

在transferForSignal方法中，首先使用cas操作设置当前节点的waitStatus为Node.CONDITION，如果设置失败说明当前节点已经被取消了，那么就直接返回，然后操作下一个节点；如果修改成功，说明该节点已经在条件队列中被唤醒，但是还没有获取到锁，所以需要转移到等待队列的末尾等待获取锁。  

在节点被enq方法成功添加到等待队列的末尾后，返回的是它的前驱节点。我们知道等待队列中的节点都需要靠前驱节点来唤醒，所以这里需要做就是将前驱节点的waitStatus设置为Node.SIGNAL。如果前驱节点的状态是取消状态或者设置为Node.SIGNAL失败，那么就直接唤醒当前节点。这时锁不一定是可用的状态，有可能线程被唤醒了但是锁还是被占用的状态，大不了获取不到锁再次被挂起。

这里总结一下signalAll方法  

+ 将条件队列清空（firstWaiter=lastWaiter=null）  
+ 将条件队列的头节点出队，使之称为孤立节点（nextWaiter、prev、next都为null）  
+ 如果该节点被取消了，则直接跳过该节点（孤立的节点会被GC回收）  
+ 如果该节点处于正常状态，则通过enq方法添加到sync queue的末尾  
+ 判断是否需要将该节点唤醒（包括设置前驱节点的状态为SIGNAL），如果有必要，直接唤醒当前节点  
+ 重复上述过程  

#### signal方法  

与signalAll方法大同小异，signal方法只会唤醒条件队列中第一个没有被取消的节点  

```java
public final void signal() {
    //判断调用signal方法的线程是不是当前线程
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    //如果条件队列不为空，唤醒条件队列中第一个没有被取消的节点
    if (first != null)
        doSignal(first);
}
```

```java
private void doSignal(Node first) {
    do {
        //将firstWaiter指向队列头的下一个节点
        if ( (firstWaiter = first.nextWaiter) == null)
            lastWaiter = null;
        //将原来队列中的头节点从队列中断开，使之成为一个孤立节点
        first.nextWaiter = null;
    } while (!transferForSignal(first) &&
             (first = firstWaiter) != null);
}
```

这里依然调用transferForSignal方法节点转移到等待队列的末尾。如果节点被取消，那么transferForSignal方法会返回false，则循环下一个节点；如果没有被取消，那么transferForSignal方法就会返回true，此时while条件不满足，就会退出循环，方法就会结束。因此调用signal方法，只会唤醒一个线程。  

#### await方法  

在调用signal或者signalAll方法之后，会将节点接入到等待队列的末尾，要么立即唤醒线程，要么等待前驱节点唤醒线程，总之唤醒的线程是要恢复执行的。那么从哪里恢复执行呢？答案还是在调用await的方法中  

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

线程被唤醒的原因我们是不是知道的，有可能是因为其他线程执行了signal，也有可能是因为中断，但不论什么原因，被唤醒的线程需要离开条件队列进入到等待队列中。

之后通过acquireQueued方法进行阻塞式的抢占锁，如果抢占到了锁就返回；如果没有抢占到锁，就继续被挂起。因此await方法返回时，线程一定是抢占到了锁。  

有一点需要提前说明一下，那就是如果线程被唤醒到线程获取到锁的这段过程中发生过中断怎么办？  

之前说过，中断对于线程只是一个建议，由线程自己决定怎么对其进行处理。在acquireQueued方法中，我们对中断时不响应的，即使简单记录中断的状态，并在抢到锁之后将这个状态返回，交给上层调用的函数处理，这里就是交给await方法来处理  

那么await怎么处理这个中断呢？这取决于：中断发生时，线程是否被signal过  

如果中断发生时，当前线程没有被signal过，则说明当前线程还处于条件队列中，属于正常在等待中的状态，此时中断将导致当前线程从正常等待行为被打断，进入到等待队列中抢占锁。因此从await方法返回后，则需要抛出InterruptedException，表示当前线程因为中断而被唤醒。  

如果中断发生时，当前线程已经被signal过了，则说明这个中断来的太晚了，既然当前线程已经被signal过了，说明中断发生时，当前线程已经在条件队列中被正常唤醒了，所以即使发生中断，也需要等待await方法返回时，再自我中断弥补一下这个中断。

await使用interruptMode来记录中断事件，该变量有三个值  

1. 0：代表整个过程中没有发生中断  
2. THROW_IE：表示退出await方法时需要抛出InterruptedException异常，这种模式对应中断发生在signal之前  
3. REINTERRUPT：表示退出await方法时需要再自我中断一下，这种模式对应于中断发生在signal之后  

```java
/** Mode meaning to reinterrupt on exit from wait */
private static final int REINTERRUPT =  1;
/** Mode meaning to throw InterruptedException on exit from wait */
private static final int THROW_IE    = -1;
```

线程被唤醒后，首先使用checkInterruptWhileWaiting方法来检测中断的模式  

```java
private int checkInterruptWhileWaiting(Node node) {
    return Thread.interrupted() ?
        (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) :
    0;
}
```

如果没有发生过中断，Thread.interrupt返回false，则checkInterruptWhileWaiting返回0；如果已经发生过中断，则Thread.interrupted返回true，接下来就是使用transferAfterCancelledWait来判断是否发生了signal  

```java
final boolean transferAfterCancelledWait(Node node) {
    if (compareAndSetWaitStatus(node, Node.CONDITION, 0)) {
        enq(node);
        return true;
    }
    /*
    * If we lost out to a signal(), then we can't proceed
    * until it finishes its enq().  Cancelling during an
    * incomplete transfer is both rare and transient, so just
    * spin.
    */
    while (!isOnSyncQueue(node))
        Thread.yield();
    return false;
}
```

判断一个node是否被signal，一个简单有效的方法就是判断它是否离开了条件队列，进入到等待队列中。  

换句话说，只要一个节点的waitStatus还是Node.CONDITION，那就说明它还没有被signal过。  

##### 情况一  

假设中断发生在signal之前，则上面的方法中compareAndSetWaitStatus会返回true，然后通过enq将节点加入到等待队列的末尾，然后返回true。需要注意的是此时还没有断开node节点的nextWaiter指针，所以后面还需要断开。  

回到transferAfterCancelledWait的调用处，可以知道checkInterruptWhileWaiting方法返回THROW_IE，也就是interruptMode=-1，此时回到checkInterruptWhileWaiting的调用出  

```java
while (!isOnSyncQueue(node)) {
    LockSupport.park(this);//当前线程在这里被挂起，后面被唤醒以后，同样从这里开始执行


    //线程执行到这里说明要么是被signal方法唤醒，要么是线程被中断
    //所以下面检查中断状态，如果是被中断，则跳出while循环
    //checkInterruptWhileWaiting判断是否有中断发生以及中断发生在signal调用之前还是之后
    if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
        break;
}
if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
    interruptMode = REINTERRUPT;
//移除条件队列中所有已经被取消的节点
if (node.nextWaiter != null) // clean up if cancelled
    unlinkCancelledWaiters();
//汇报中断状态
if (interruptMode != 0)
    reportInterruptAfterWait(interruptMode);
```

因为此时interruptMode=-1，所以执行break，从while循环中退出，然后执行acquireQueued争抢锁。注意这里传入的

savedState是之前fullyRelease返回的值。  

假设此时acquireQueued获取到了锁，但是不满足interruptMode != THROW_IE，所以跳过第一个if中逻辑，进入到  

```java
if (node.nextWaiter != null) // clean up if cancelled
    unlinkCancelledWaiters();
```

前面说过当前节点的nextWaiter还是有值，它并没有和原来的条件队列断开连接，这里已经获取到了锁，所以从等待队列中移除了，接下来就需要从条件队列中移除。既然是从头遍历，那么索性就将条件队列中所有已经取消的节点都移除。  

节点被移除后，接下来就是最后一步汇报中断状态  

```java
if (interruptMode != 0
    reportInterruptAfterWait(interruptMode);
```

这里interruptMode=THROW_IE，说明发生了中断，则调用reportInterruptAfterWait  

```java
private void reportInterruptAfterWait(int interruptMode)
    throws InterruptedException {
    if (interruptMode == THROW_IE)
        throw new InterruptedException();
    else if (interruptMode == REINTERRUPT)
        selfInterrupt();
}
```

可以看出interruptMode == THROW_IE会简单的抛出一个InterruptedException异常  

总结  

1. 线程因为中断，从挂起的地方被唤醒  

2. 随后，通过transferAfterCancelledWait方法确认了线程的waitStatus为Node.CONDITION，说明中断发生时线程没有被signal      

3. 然后修改线程的waitStatus为0，并通过enq方法将其添加到等待队列中  

4. 接下来线程将在等待队列中等待获取锁，如果获取不到锁，会再次被挂起  

5. 线程获取到锁之后，将调用unlinkCancelledWaiters方法将节点从条件队列中移除，该方法还会顺便移除队列中其他的被取消的节点  

6. 最后通过reportInterruptAfterWait抛出一个InterruptedException异常  

可以看出，一个调用了await方法的挂起的线程在被中断后不会立即抛出InterruptedException异常，而是会被添加到等待队列中去争抢锁。如果没有争抢到锁，还是会被挂起的；如果争抢到了，则会从等待队列和条件队列中移除，然后才抛出InterruptedException异常  

##### 情况二   

中断发生时，线程已经被signal了。  

其实这种情况包含了两种子情况  

1. 被唤醒时，已经发生过了中断，但是此时线程已经被signal过了  
2. 被唤醒时，没有发生中断，在争抢锁的过程中发生了中断  

###### 子情况一  

被唤醒时，已经发生过了中断，但是此时线程已经被signal过了  

对于这种情况，和前面的情况的差别主要在于transferAfterCancelledWait方法  

```java
final boolean transferAfterCancelledWait(Node node) {
    if (compareAndSetWaitStatus(node, Node.CONDITION, 0)) {
        enq(node);
        return true;
    }
    /*
    * If we lost out to a signal(), then we can't proceed
    * until it finishes its enq().  Cancelling during an
    * incomplete transfer is both rare and transient, so just
    * spin.
    * 发生中断前已经被signal过了，只需要等待节点被加入到等待队列
    * 节点有可能已经进入到等待队列或者在进入等待队列的路上，所以使
    * 用自旋的方式，直到节点进入到等待队列
    */
    while (!isOnSyncQueue(node))
        Thread.yield();
    return false;
}
```

在这里由于signal已经发生过了，所以节点的waitStatus必不为Node.CONDITION，所以将跳过if语句。此时线程可能已经进入等待队列或者在进入等待队列的路上。  最终返回false。  

返回到transferAfterCancelledWait的调用处，checkInterruptWhileWaiting方法返回REINTERRUPT，这说明我们在退出await时需要再次自我中断一下。  

返回到checkInterruptWhileWaiting的调用处  

```java
while (!isOnSyncQueue(node)) {
    LockSupport.park(this);//当前线程在这里被挂起，后面被唤醒以后，同样从这里开始执行


    //线程执行到这里说明要么是被signal方法唤醒，要么是线程被中断
    //所以下面检查中断状态，如果是被中断，则跳出while循环
    //checkInterruptWhileWaiting判断是否有中断发生以及中断发生在signal调用之前还是之后
    if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
        break;
}
if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
    interruptMode = REINTERRUPT;
//移除条件队列中所有已经被取消的节点
if (node.nextWaiter != null) // clean up if cancelled
    unlinkCancelledWaiters();
//汇报中断状态
if (interruptMode != 0)
    reportInterruptAfterWait(interruptMode);
```

此时interruptMode不等于0，直接跳出while循环，接下来就是和上面一样在acquireQueued方法中争抢锁，成功则退出；不成功则挂起。最后一定会获取锁然后退出acquireQueued方法。接下来就是将节点从条件队列中移除，不过与情况一不同，这时节点的nextWaiter已经被设置为null，所以这里不需要执行。最后就是向上汇报中断状态。  

```java
private void reportInterruptAfterWait(int interruptMode)
    throws InterruptedException {
    if (interruptMode == THROW_IE)
        throw new InterruptedException();
    else if (interruptMode == REINTERRUPT)
        selfInterrupt();
}
```

此时由于interruptMode == REINTERRUPT，所以只需要执行selfInterrupt，将当前线程中断一次  

总结  

1. 线程从挂起的地方被唤醒，此时既发生过中断，又发生过signal  

2. 随后在transferAfterCancelledWait中判断中断发生在signal调用之后  

3. 然后通过自旋的方式等待signal完成，确保节点已经成功添加到等待队列中去  

4. 接下来在等待队列中继续争抢锁，争抢不到会被继续挂起  

5. 线程成功获取到锁以后，通过reportInterruptAfterWait方法将线程再次中断而不是抛出InterruptedException异常  

###### 子情况二  

被唤醒时没有发生中断，但是在争抢锁的过程中发生中断  

如果线程是因为signal被唤醒，则由前面分析signal的方法可以知道，线程最终都会离开条件队列，加入到等待队列末尾。所以只需要判断被唤醒时，线程是否已经在等待队列中即可  

```java
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    Node node = addConditionWaiter();
    int savedState = fullyRelease(node);
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);  // 我们在这里，线程将在这里被唤醒
        // 由于现在没有发生中断，所以interruptMode目前为0
        //由于中断发生在争抢锁的时候，所以唤醒时还没有发生中断，因此checkInterruptWhileWaiting
        //返回0，在下一次循环中，节点已经在sync queue中了，所以退出循环
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

由于中断发生在争抢锁的过程中，所以被唤醒时线程是没有被中断的。所以checkInterruptWhileWaiting方法返回了0，在下一次循环中，由于线程已经被唤醒了，所以在等待队列中了，因此就退出了while循环。所以这种情况下，interruptMode=0。

接下来在acquireQueued中争抢锁，这个方法仅仅会记录一下线程的中断状态，最后争抢锁成功以后会返回true，则此时interruptMode会被设置为REINTERRUPT。之后由于在signal方法的时候节点已经从条件队列中被移除了，所以node.nextWaiter != null条件就不成立了，所以到最后就是reportInterruptAfterWait汇报自己的中断状态。

总结  

+ 线程被signal方法唤醒后，此时并没有发生过中断。  

+ 因为没有发生过中断，所以从checkInterruptWhileWaiting返回时interruptMode=0。

+ 接下来回到循环中，因为signal方法使得节点被转移到了等待队列中，所以while条件不满足，退出循环。

+ 然后在acquireQueued中以阻塞的方式争抢锁，如果没有争抢到锁则被挂起。  

+ 线程获取锁返回后，检测到在争抢锁的过程中发生过中断，并且此时interruptMode=0，所以这时将interruptMode设置为 REINTERRUPT。  

+ 最后通过reportInterruptAfterWait将当前线程再次中断，但是不会抛出InterruptedException异常。  

不管怎么说，只要中断发生在signal之后，我们都认为中断来的太晚了，在await中都将忽略这个中断，只是在await方法返回的时候，将这个线程再中断一次，而不是抛出异常。  

##### 情况三  

一直没有发生过中断  

这种情况就不需要汇报中断了，所以直接从await方法返回就可以了  

#### await方法总结  

1. 进入和离开await方法必须是已经持有了锁  

2. 调用await方法会使得当前线程被封装成一个Node节点加入到条件队列中，这是一个靠nextWaiter连接的单向链表，然后释放所持有的锁  

3. 释放锁以后，当前线程会在条件队列中被挂起，直到被signal唤醒或者中断  

4. 线程抢到锁以后，会从条件队列中转移到等待队列中，进行锁的争抢

5. 若在线程获得锁之前发生了中断，需要根据中断发生在signal之前还是之后记录中断模式，做不同的处理  

6. 线程在抢到锁以后进行善后工作，包括离开条件队列，处理中断  

7. 线程持有了锁，从await方法中离开  

![preview](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/1460000016462286)  

如前所说，signal和中断都可以将线程从条件队列移除添加到等待队列中去争抢锁，所不同的是,signal方法被认为是正常的唤醒线程，中断被认为是不正常的唤醒线程。如果中断发生在signal之前，则在最终返回时需要抛出InterruptedException异常；如果中断发生在signal之后，认为这个线程已经被正常唤醒，所以只是在await返回时自我中断一下，相当于将这个中断推迟到await方法返回时在发生。  

#### awaitUninterrupibly方法  

await方法中，中断起到了和signal同样的效果，但是中断属于将一个等待中的线程非正常的唤醒，可能即使线程被唤醒后，也抢到了锁，但是却发现当前的条件不满足，这样的情况还需要将线程挂起。因此可能有时并不希望await方法被中断，awaitUninterruptibly方法则实现了这个功能  

```java
public final void awaitUninterruptibly() {
    //包装成node节点加入到条件队列
    Node node = addConditionWaiter();
    //完全释放线程持有的锁
    int savedState = fullyRelease(node);
    boolean interrupted = false;
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);//线程在这里被挂起，唤醒后继续从这里执行
        if (Thread.interrupted())
            //发生中断后线程依然在条件队列中，会在循环中再次被挂起，仅仅是记录一下线程的中断状态
            interrupted = true;
    }
    if (acquireQueued(node, savedState) || interrupted)
        selfInterrupt();
}
```

可以看到awaitUninterruptibly和await方法的区别在于，await方法中的checkInterruptWhileWaiting方法会记录中断发生在signal之前还是之后，然后将节点从条件队列转移到等待队列中；而在awaitUninterruptibly方法中，即使挂起的线程被中断唤醒，也仅仅是记录一下发生过中断，然后再循环中再次将线程挂起，除非是发生了signal或者signalAll。  

它的核心思想就是  

1. 中断虽然会唤醒线程，但是不会导致线程离开条件队列，如果线程只是因为中断而被唤醒，则它将再次被挂起  

2. 中有signal方法或者signalAll方法会使得线程离开条件队列  

3. 调用该方法时或者调用过程中如果发生了中断，仅仅会在方法结束的时候再自我中断一下而不是抛出InterruptedException异常  

#### awaitNanos方法  

无论是await还是awaitUninterruptibly方法，它们在抢占锁的过程中都是阻塞式的，即一直到抢到了锁为止，否则线程还是会被挂起，这样有一个问题就是如果线程长时间没有抢占到锁就会一直被阻塞，因此有时需要带有超时机制的抢锁  

```java
public final long awaitNanos(long nanosTimeout)
    throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    //将线程包装成node节点加入到条件队列中
    Node node = addConditionWaiter();
    //完全释放线程上的锁
    int savedState = fullyRelease(node);
    //计算超时的截止时间
    final long deadline = System.nanoTime() + nanosTimeout;
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        if (nanosTimeout <= 0L) {
            transferAfterCancelledWait(node);
            break;
        }
        if (nanosTimeout >= spinForTimeoutThreshold)
            LockSupport.parkNanos(this, nanosTimeout);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
        nanosTimeout = deadline - System.nanoTime();
    }
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null)
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
    return deadline - System.nanoTime();
}
```

