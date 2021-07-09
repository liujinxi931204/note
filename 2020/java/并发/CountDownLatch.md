CountDownLatch是一个很有用的工具，该工具是为了解决某些操作只能在一组操作全部执行完成后才能执行的场景。例如，小组早上开会，只有等所有人都到齐了才能开始。CountDownLatch是倒数计数器，所以CountDownLatch的用法通常是设定一个大于0的值，该值表示需要等待的总任务数，每完成一个任务，将总任务数减一，直到最后总任务数为0，说明所有的任务都完成了，后面的任务可以继续了    

## 核心属性  

CountDownLatch主要是通过AQS的共享锁机制实现，因此它的核心属性只有一个sync，继承自AQS，同时重写了tryAcquireShared和tryReleaseShared，已完成具体的实现共享锁的获取与释放逻辑  

```java
private static final class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = 4982264981922014374L;

    Sync(int count) {
        setState(count);
    }

    int getCount() {
        return getState();
    }

    protected int tryAcquireShared(int acquires) {
        return (getState() == 0) ? 1 : -1;
    }

    protected boolean tryReleaseShared(int releases) {
        // Decrement count; signal when transition to zero
        for (;;) {
            int c = getState();
            if (c == 0)
                return false;
            int nextc = c-1;
            if (compareAndSetState(c, nextc))
                return nextc == 0;
        }
    }
}
```

## 构造函数  

```java
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
```

在构造函数中，就是简单的传入一个任务数，如果任务数小于0，会抛出异常；如果大于等于0，调用内部类Sync的构造方法，就是设置AQS中的state的初始值  

## 核心方法  

CountDownLatch有两个核心方法，一个是countDown方法，每调用一次，就会将当前的count减一，当count的值为0，就会唤醒所有等待的线程；另一个方法是await方法，该方法用于将当前线程挂起，直到count的值为0。虽然和条件队列的用法很像，但是CountDownLatch底层用的是共享锁而不是条件队列  

### countDown 方法  

```java
public void countDown() {
    sync.releaseShared(1);
}
```

这个方法的目的就是将count的值减一，并且在count值等于0的时候，唤醒所有等待的线程，它的内部其实就是调用共享锁的释放操作  

```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

这个方法由AQS实现，但是tryReleaseShared方法由Sync内部类自己实现  

```java
protected boolean tryReleaseShared(int releases) {
    // Decrement count; signal when transition to zero
    for (;;) {
        int c = getState();
        if (c == 0)
            return false;
        int nextc = c-1;
        if (compareAndSetState(c, nextc))
            return nextc == 0;
    }
}
```

这个方法就是获取当前的state值，如果已经为0了，直接返回false；否则通过cas操作将state值减一，之后返回nextc == 0。从这里可以看到，如果count的值从不是0变为0，才会返回true，其他的情况都会返回false。而且如果count的值已经成为0了，再次调用这个方法的时候还是会返回false。也就是这个方法其实只关心一种情况，就是state的值从不是0变为0的这种情况，这时代表倒数计数结束，所有被阻塞的任务都应该继续进行了  

doReleaseShared方法这里不详细说，主要的作用就是唤醒所有等待中的线程  

### await方法  

与condition的await方法的语义相同，该方法是阻塞式地等待，并且是响应中断的，只不过它不是在等待signal操作，而是在等待count的值为0  

```java
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```

可见在await方法内部调用了AQS的acquireSharedInterruptibly方法  

```java
public final void acquireSharedInterruptibly(int arg)
    throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}
```

可以看到这里需要调用AQS的子类Sync实现的tryAcquireShared方法  

```java
protected int tryAcquireShared(int acquires) {
    return (getState() == 0) ? 1 : -1;
}
```

发现这里和获取锁没有什么太大的关系，就是判断当前的state值是不是0，如果是0，就返回1；否则就返回-1。可以知道，当count值不为0的时候，调用await方法的线程需要被阻塞；如果count值为0，那么调用await方法的线程就不应该被阻塞而是继续向下运行。 也就可以知道， doAcquireSharedInterruptibly方法其实就是将当前线程包装成Node节点然后放入等待队列  

```java
private void doAcquireSharedInterruptibly(int arg)
    throws InterruptedException {
    //将当线程变为Node节点，然后加入到等待队列中
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

### await(long timeout,TimeUnit unit)方法  

相比较于await方法，await(long timeout,TimeUnit unit)方法提供了超时的机制  

```java
public boolean await(long timeout, TimeUnit unit)
    throws InterruptedException {
    return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
}
```

可见其内部调用了Sync的tryAcquireSharedNanos方法  

```java
public final boolean tryAcquireSharedNanos(int arg, long nanosTimeout)
    throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    return tryAcquireShared(arg) >= 0 ||
        doAcquireSharedNanos(arg, nanosTimeout);
}
```

tryAcquireShared上面已经说过了，就是判断当前的state值是不是0。如果不是0，返回-1。说明此时调用doAcquireSharedNanos方法，需要阻塞当前线程一段时间  

```java
private boolean doAcquireSharedNanos(int arg, long nanosTimeout)
    throws InterruptedException {
    if (nanosTimeout <= 0L)
        return false;
    final long deadline = System.nanoTime() + nanosTimeout;
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return true;
                }
            }
            nanosTimeout = deadline - System.nanoTime();
            if (nanosTimeout <= 0L)
                return false;
            if (shouldParkAfterFailedAcquire(p, node) &&
                nanosTimeout > spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);
            if (Thread.interrupted())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```





