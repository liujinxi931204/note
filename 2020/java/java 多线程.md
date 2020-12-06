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
**使用Runnable接口的方式实现多线程可以避免Java的单继承问题，而且实现Runnable接口的方式更容易实现数据的共享，此外Thread本质上也是实现了Runnable接口的**  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/04/1607075667626-1607075667628.png)  
**可以看到Thread类也是继承了Runnable接口的**  
  
如下是一个简单实现Runnable接口实现数据共享  
```java
package com.sogou;


public class threadSecond implements Runnable {
    
    private int i=100;

    @Override
    public void run() {
        while(i>0){
            System.out.println(Thread.currentThread().getName()+"   "+--i);
        }
    }
    public static void main(String[] args) {
        threadSecond threadSecond = new threadSecond();
        new Thread(threadSecond).start();
        new Thread(threadSecond).start();
        System.out.println(Thread.currentThread().getName());

    }
}

```
### 常见错误：调用run()方法而非start()方法  
创建并运行一个线程所犯的常见错误是调用线程的run()方法而非start()方法  
Thread.java 类中的start()方法通知“线程规划器”此线程已经准备就绪，等待调用线程对象的run()方法  
```java
Thread newThread=new Thread(myRunnable);
newThread.run()；//应该是start()
```  
事实上，run()方法并非是由刚创建的新线程所执行的，而是由被创建现场的当前线程所执行了。也就是由被执行上述两行代码的线程所执行  
### 线程控制的方法  
#### currentThread  
currentThread()方法是Thread类的静态方法，直接使用Threadj就可以使用该方法，该方法可以返回代码段正在被哪个线程调用的信息  
```java
package com.sogou;


public class threadThird implements Runnable {
    @Override
    public void run() {
        //打印当前线程的线程名
        System.out.println(Thread.currentThread().getName());
    }


    public static void main(String[] args) {
        threadThird threadThird = new threadThird();
        new Thread(threadThird,"线程1").start();
        new Thread(threadThird, "线程2").start();    
        System.out.println(Thread.currentThread().getName());
    }
}
```  
#### isAlive  
当调用Thread类的start()方法并且没有线程尚未死亡时，线程被任务是活着的  
```java
package com.sogou;


public class threadFourth  implements Runnable{
    @Override
    public void run() {
        try {
            Thread.sleep(30);
            System.out.println("当前线程是否存活"+Thread.currentThread().isAlive());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }


    public static void main(String[] args) {
        threadFourth threadFourth = new threadFourth();
        new Thread(threadFourth).start();
    }
}

```  
### sleep  
Thread.sleep()方法是Thread类的静态方法，是当前线程进入休眠状态，这时会交出CPU的使用权，如果线程在sleep状态被中断，将会抛出IterruptedException中断异常  
sleep()方法只是线程交出CPU使用权，不代表会立即重新获得使用权   
```java
package com.sogou;

public class threadFifth implements Runnable {
    @Override
    public void run() {
        try {
            for (int i = 0; i <5 ; i++) {
                System.out.println(i);
                Thread.sleep(30);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        threadFifth threadFifth = new threadFifth();
        new Thread(threadFifth).start();
        new Thread(threadFifth).start();
    }
}
```  
**sleep()方法是不会释放所持有的锁**  
```java
package com.sogou;

public class threadFifth implements Runnable {
    @Override
    public void run() {
        synchronized (this){
            try {
                for (int i = 0; i <5 ; i++) {
                    System.out.println(i);
                    Thread.sleep(30);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        threadFifth threadFifth = new threadFifth();
        new Thread(threadFifth).start();
        new Thread(threadFifth).start();
    }

}
```  
上述两个方法的输出结果分别是  
```java
0011223344
0123401234
```
出现这样就是因为sleep的时候不会释放当前线程的锁  
### 停止线程  
停止线程不像break那样

