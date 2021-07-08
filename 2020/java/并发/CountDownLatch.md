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

