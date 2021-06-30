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

![CLH队列](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/CLH%E9%98%9F%E5%88%97.png)

sync queue是一个双向链表，使用prev、next属性来串联节点。但是在同步队列中，始终没有用到nextWaiter属性，即使是在共享模式下，这一属性也仅仅是作为一个标志，指向了一个空节点。因此，在sync queue中，不会使用nextWaiter属性来串联节点。  

