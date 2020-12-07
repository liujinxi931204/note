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
停止线程不像break那样简单粗暴，需要一些技巧性的处理  
有以下3种方式  
+ 使用退出标志，使线程正常退出，也就是当run()方法执行完以后  
+ 使用stop()方法强行终止，但是不推荐这个方法，这个方法已经被放弃了，使用以后可能会产生不可预料的后果  
+ 使用interput()方法中断线程  
#### 使用退出标志  
当run()方法执行以后，线程就会退出。但有时run()方法是永远不会结束的，如果在服务端程序中使用线程进行客户端监听请求，或是其他的需要循环处理的任务  
在这种情况下，一般是将这些任务放在一个循环里，如while循环。如果想使while循环在某一特定条件下退出，最直接的方法就是设置一个boolean类型的标志位，并通过这个标志位true或者false来控制while循环是否退出  
```java
public class test1 {

    public static volatile boolean exit =false;  //退出标志
    
    public static void main(String[] args) {
        new Thread() {
            public void run() {
                System.out.println("线程启动了");
                while (!exit) {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println("线程结束了");
            }
        }.start();
        
        try {
            Thread.sleep(1000 * 5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        exit = true;//5秒后更改退出标志的值,没有这段代码，线程就一直不能停止
    }
}
```  
#### 使用interupt方法  
Tread.interrupt()方法：作用是中断线程。将会设置线程的中断状态位，即设置位true，中断的结果是线程死亡、还是等待新的任务或者是继续运行到下一步，就取决于这个程序本身。线程会不时地检测这个中断标志位，以判断线程是否应该被中断(中断标识是否位true)。它并不像stop()方法那样中断一个正在运行地线程  
interrupt()方法**只是改变中断状态，不会中断一个正在运行地线程**。需要用户自己去监视线程的状态。支持线程中断的方法(也就是线程中断后会抛出interruptedException的方法)就是在**监视线程的中断状态，一旦线程的中断状态被置为"中断状态"，就会抛出异常**。这一方法实际完成的是，给受阻塞的线程发出一个中断信号，这一受阻塞的线程检查到中断标识，就得以退出阻塞状态  
如果线程被Object.wait()、Thread.join()和Thread.sleep()三种方法之一阻塞，此时调用该线程的interrput()方法，那么该线程将抛出一个interruptedExection中断异常(该线程必须提前处理好此异常)，从而提早地终结被阻塞状态。如果线程没有被阻塞，这时调用interrupt(）将不起作用，直到执行到wait()、sleep()、join()

