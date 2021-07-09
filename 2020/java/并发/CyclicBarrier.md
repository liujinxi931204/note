## 与CountDownLatch的区别  

### 将count值递减的线程  

在CountDownLatch中，执行countDown和await的线程不是同一个线程。执行CountDownLatch的线程不会被挂起，调用await方法的线程会被挂起等待共享锁。而在CyclicBarrier中，将count值递减的行为和执行await方法的线程是同一个线程，他们在执行完递减操作后，如果count的值不为0，可能同时会被挂起

### 是否重复使用  

CountDownLatch是一次性的，当count值被减为0后，不会被重置；CyclicBarrier在线程通过栅栏后，会开启新的一代，count值会被重置  

### 锁的类别与所用到的队列  

CountDownLatch使用的是共享锁，count值不为0时，线程在等待队列中等待。由于使用共享锁，唤醒操作不必等待释放锁之后再进行，唤醒操作很迅速；CyclicBarrier使用的是独占锁，count值不为0时，线程进入条件队列中等待，当count值为0后，将调用signallAll方法唤醒所有线程到等待队列中去争抢锁，在前驱节点释放锁以后，才能唤醒后继节点  

