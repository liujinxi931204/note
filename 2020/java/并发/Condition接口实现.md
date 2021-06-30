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
    if (t != null && t.waitStatus != Node.CONDITION) {
        unlinkCancelledWaiters();
        t = lastWaiter;
    }
    Node node = new Node(Thread.currentThread(), Node.CONDITION);
    if (t == null)
        firstWaiter = node;
    else
        t.nextWaiter = node;
    lastWaiter = node;
    return node;
}
```



