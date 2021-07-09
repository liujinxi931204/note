信号量Semaphore也是常用的并发工具之一，它常常用于流量控制。使用Semaphore可以帮助我们有效的管理

Semaphore的结构和ReentrantLock以及CountDownLatch很像，内部采用了公平锁和非公平锁两种实现  

## 核心属性  

与CountDownLatch类似，Semaphore主要是通过AQS的共享锁来实现的，因此它的核心属性就是Sync，继承自AQS  

```java
private final Sync sync;

abstract static class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = 1192457210091910933L;

    Sync(int permits) {
        setState(permits);
    }

    final int getPermits() {
        return getState();
    }

    final int nonfairTryAcquireShared(int acquires) {
        //省略
    }

    protected final boolean tryReleaseShared(int releases) {
        //
    }

    final void reducePermits(int reductions) {
        //省略
    }

    final int drainPermits() {
        //省略
    }
}
```

这里permits和CountDownLatch的count很像，它们最终都将称为AQS的state属性的初始值  

## 构造函数  

```java
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```

默认的构造函数使用的是非公平锁，另一个构造函数通过传入的fair参数来决定使用公平锁还是非公平锁，这一点和ReentrantLock用的是同样的套路  

公平锁和非公平锁的定义如下  

```java
static final class FairSync extends Sync {
    
   FairSync(int permits) {
        super(permits);
    }

    protected int tryAcquireShared(int acquires) {
        for (;;) {
            if (hasQueuedPredecessors())
                return -1;
            int available = getState();
            int remaining = available - acquires;
            if (remaining < 0 ||
                compareAndSetState(available, remaining))
                return remaining;
        }
    }
}

static final class NonfairSync extends Sync {
    
   NonfairSync(int permits) {
        super(permits);
    }

    protected int tryAcquireShared(int acquires) {
        return nonfairTryAcquireShared(acquires);
    }
}

```

## 获取信号量  

获取信号量的方法有4个  

|             acquire方法             |                 本质调用                 |
| :---------------------------------: | :--------------------------------------: |
|              acquire()              |    sync.acquireSharedInterruptibly(1)    |
|        acquire(int permits)         | sync.acquireSharedInterruptibly(permits) |
|      acquireUninterruptibly()       |          sync.acquireShared(1)           |
| acquireUninterruptibly(int permits) |       sync.acquireShared(permits)        |

可见，调用acquire()方法就相当于调用了acquire(1)。这些方法的返回值代表的是剩余的信号量的值，如果返回值为负数，说明信号量已经不够了  

## 非公平锁实现  

```java
protected int tryAcquireShared(int acquires) {
    return nonfairTryAcquireShared(acquires);
}
```

```java
final int nonfairTryAcquireShared(int acquires) {
    for (;;) {
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 || 
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```

可以看到tryAcquireShared方法内部调用的是nonfairTryAcquireShared方法。该方法用自旋+CAS操作的方式修改state值，退出该方法的办法是成功修改了state的值或者扣除本次获取的信号量之后剩余的信号量小于0。