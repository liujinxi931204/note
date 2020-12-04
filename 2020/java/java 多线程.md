## 概念  
### 进程  
+ 正在运行的程序，是系统进行资源分配和调用的独立单位  
+ 每一个进程都有它自己的内存空间和系统资源，一个进程包括由操作系统分配的内存空间，包含一个或多个线程  
+ 一个进程一直运行，直到所有的非守护线程都结束运行后才能结束  
### 线程  
+ 线程是进程中的单个顺序控制流，是一条执行路径  
+ 一个进程如果只有一条执行路径，则称为"单线程程序"  
+ 一个进程如果由多条执行路径，则称为"多线程程序"  
+ 一个线程不能独立的存在，它必须是进程的一部分  
+ 线程是CPU调度的最小单位  
### 线程的状态  
线程分为五个阶段  
+ 创建(new)状态：准备好了一个多线程的对象  
+ 就绪(runnable)状态：调用了start()方法，它的状态变为runnable状态。控制权又被给予线程调度来完成它的执行。是否立即运行此线程火灾运行之前将其保持在可运行线程池中，取决于线程调度的实现，等待CPU进行调度  
+ 运行(running)状态：执行run()方法，线程正在执行。线程调度程序可运行线程池中选择一个线程，并将其状态更改为正在运行，然后CPU开始执行这个线程  
+ 阻塞(blocked)状态：暂时停止执行，可能将资源交给其他线程使用  
+ 终止(dead)状态：一旦线程完成窒息感，它的状态就变成dead，线程销毁  
### 线程的状态转换  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/04/1607073222653-1607073222658.png)  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/04/1607073267197-1607073267198.png)  
## 使用多线程  
实现多线程主要有两种方法：一种是继承Thread类，另一种是实现Runnable接口  
### 继承Thread类  
```java
package com.sogou;

public class threadFirst extends Thread{
    //继承Thread类，重写run()方法，run()中是需要为多线程执行的任务
    @Override
    public void run() {
        super.run();
        System.out.println(Thread.currentThread().getName());
    }
    public static void main(String[] args) {
        threadFirst threadFirst = new threadFirst();
        //使用start()方法会是线程进入可执行状态，具体是否会执行还需要CPU的调度
        threadFirst.start();
        System.out.println(Thread.currentThread().getName());
    }
}
```
### 实现Runnable接口  
```java
package com.sogou;


public class threadSecond implements Runnable {
    //实现Runnable接口，需要重写其中的run方法，run()中是需要为多线程执行的任务
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }

    public static void main(String[] args) {
        threadSecond threadSecond = new threadSecond();
        //创建一个Thread对象，将实现了Runnable接口的对象传给Thread对象，调用Thread对象的start()方法
        Thread thread = new Thread(threadSecond);
        thread.start();

        System.out.println(Thread.currentThread().getName());
    }
}
```


