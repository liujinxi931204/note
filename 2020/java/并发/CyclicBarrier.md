## 与CountDownLatch的区别  

### 将count值递减的线程  

在CountDownLatch中，执行countDown和await的线程不是同一个线程。执行CountDownLatch的线程不会被挂起，调用await方法的线程会被挂起等待共享锁。而在CyclicBarrier中，将count值递减的行为和执行await方法的线程是同一个线程，他们在执行完递减操作后，如果count的值不为0，可能同时会被挂起

### 是否重复使用  

CountDownLatch是一次性的，当count值被减为0后，不会被重置；CyclicBarrier在线程通过栅栏后，会开启新的一代，count值会被重置  

### 锁的类别与所用到的队列  

CountDownLatch使用的是共享锁，count值不为0时，线程在等待队列中等待。由于使用共享锁，唤醒操作不必等待释放锁之后再进行，唤醒操作很迅速；CyclicBarrier使用的是独占锁，count值不为0时，线程进入条件队列中等待，当count值为0后，将调用signallAll方法唤醒所有线程到等待队列中去争抢锁，在前驱节点释放锁以后，才能唤醒后继节点  

## 核心属性  

```java
private static class Generation {
    boolean broken = false;
}

/** The lock for guarding barrier entry */
private final ReentrantLock lock = new ReentrantLock();
/** Condition to wait on until tripped */
private final Condition trip = lock.newCondition();
/** The number of parties */
private final int parties;
/* The command to run when tripped */
private final Runnable barrierCommand;
/** The current generation */
private Generation generation = new Generation();

/**
 * Number of parties still waiting. Counts down from parties to 0
 * on each generation.  It is reset to parties on each new
 * generation or when broken.
 */
private int count;
```

CycliCBarrier的核心属性有6个，可以分为3组  

第一组  

```java
private final int parties;
private int count;
```

这两个属性都是用来表征线程的数量的，parties表示的是参与的总线程数，即需要一起通过Barrier的线程数，是final类型的，一旦创建就不会再更改的；count属性和CountDownLatch中的count是一样的，代表还需要等待的线程数，初始值为parties，每当到来一个线程，count值减少一个，如果变为了0，说明所有线程都到齐了，所有线程可以一起通过barrier。通过之后，count值被重置为parties  

第二组  

```java
private final ReentrantLock lock = new ReentrantLock();
private final Condition trip = lock.newCondition();
private Generation generation = new Generation();
```

这一组代表了CyclicBarrier的实现基础，即CyclicBarrier是基于独占锁和条件队列实现的，所有相互等待的线程都会在同样的条件队列上被挂起，被唤醒后将会添加到等待队列中争抢锁，获得锁的线程继续向下进行。这里还有一个Generation对象，下面会详细说  

第三组  

```java
private final Runnable barrierCommand;
```

  这是一个Runnable对象，代表了一个任务。当所有线程都到齐后，在它们一同通过Barrier之前，就会执行这个对象的run方法。当然这个参数不是必须的，如果线程在通过Barrier之前没有什么特比需要处理的事请，可以设置为null  

## 构造函数  

```java
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}

public CyclicBarrier(int parties) {
    this(parties, null);
}
```

可以看到，如果不传入Runnable参数，其实就是将该参数设置为null。构造函数初始化了parties、count以及barrierAction三个参数。  

## 辅助方法  

首先需要理解"Generation"的概念，由于CyclicBarrier是可重复使用的，把每一个新的Barrier称为一代  

开启新的一代，需要使用nextGeneration方法  

### nextGeneration方法  

```java
private void nextGeneration() {
    // signal completion of last generation
    trip.signalAll();
    // set up next generation
    count = parties;
    generation = new Generation();
}
```

通常开启新的一代是由最后一个到达的线程来执行的。在该方法中，会唤醒当前这一代中所有在条件队列中挂起的线程，将count的值恢复为parties，以及开启新的一代  

### breakBarrier方法  

```java
private void breakBarrier() {
    // 标记broken状态
    generation.broken = true;
    // 恢复count值
    count = parties;
    // 唤醒当前这一代中所有等待在条件队列里的线程（因为栅栏已经打破了）
    trip.signalAll();
}
```

### reset方法 

```java
public void reset() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        breakBarrier();   // break the current generation
        nextGeneration(); // start a new generation
    } finally {
        lock.unlock();
    }
}
```

reset方法用于将Barrier恢复成初始状态，内部就是简单调用了breakBarrier和nextGeneration两个方法。需要注意的是，如果在执行该方法时有线程在等待在Barrier上，则它将立即返回并抛出BrokenBarrierException异常。另外一点需要注意的是，该方法执行前需要获得锁  

### await方法  

await方法是CyclicBarrier最核心的方法，他是一个集"countDown"和"阻塞等待"于一体的方法  

await方法有两种版本，一种带有超时机制，另一种不带超时机制，然而从内部来看，不带超时机制的方法最终调用的是带有超时机制的doAwait方法  

```java
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}
public int await(long timeout, TimeUnit unit)
    throws InterruptedException,
BrokenBarrierException,
TimeoutException {
    return dowait(true, unit.toNanos(timeout));
}
```

可以说，CyclicBarrier最核心的方法都是在dowait实现的  

### dowait(boolean timed, long nanos)方法  

```java
private int dowait(boolean timed, long nanos)
    throws InterruptedException, BrokenBarrierException,
TimeoutException {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        final Generation g = generation;

        if (g.broken)
            throw new BrokenBarrierException();

        if (Thread.interrupted()) {
            breakBarrier();
            throw new InterruptedException();
        }

        int index = --count;
        if (index == 0) {  // tripped
            boolean ranAction = false;
            try {
                final Runnable command = barrierCommand;
                if (command != null)
                    command.run();
                ranAction = true;
                nextGeneration();
                return 0;
            } finally {
                if (!ranAction)
                    breakBarrier();
            }
        }

        // loop until tripped, broken, interrupted, or timed out
        for (;;) {
            try {
                if (!timed)
                    trip.await();
                else if (nanos > 0L)
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                if (g == generation && ! g.broken) {
                    breakBarrier();
                    throw ie;
                } else {
                    // We're about to finish waiting even if we had not
                    // been interrupted, so this interrupt is deemed to
                    // "belong" to subsequent execution.
                    Thread.currentThread().interrupt();
                }
            }

            if (g.broken)
                throw new BrokenBarrierException();

            if (g != generation)
                return index;

            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {
        lock.unlock();
    }
}
```



