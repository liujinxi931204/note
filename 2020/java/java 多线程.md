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
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/12/04/1607073222653-1607073222658.png)  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/12/04/1607073267197-1607073267198.png)  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/12/08/1607410862795-1607410862820.png)  

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
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/12/04/1607075667626-1607075667628.png)  
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
#### sleep  
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

**sleep是静态方法，最好不用使用Thread的实例对象调用它，因为它睡眠的始终是当前正在运行的线程，而不是调用它的线程对象，它只对正在运行状态的线程对象有效**  
```java
public class Test1 {  
    public static void main(String[] args) throws InterruptedException {  
        System.out.println(Thread.currentThread().getName());  
        MyThread myThread=new MyThread();  
        myThread.start();  
        // 这里sleep的就是main线程，而非myThread线程 
        myThread.sleep(1000); 
        Thread.sleep(10);  
        for(int i=0;i<100;i++){  
            System.out.println("main"+i);  
        }  
    }  
}  

```
#### 停止线程  
停止线程不像break那样简单粗暴，需要一些技巧性的处理  
有以下3种方式  
+ 使用退出标志，使线程正常退出，也就是当run()方法执行完以后  
+ 使用stop()方法强行终止，但是不推荐这个方法，这个方法已经被放弃了，使用以后可能会产生不可预料的后果  
+ 使用interput()方法中断线程  
##### 使用退出标志  
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
##### 使用interupt方法  
Tread.interrupt()方法：作用是中断线程。将会设置线程的中断状态位，即设置位true，中断的结果是线程死亡、还是等待新的任务或者是继续运行到下一步，就取决于这个程序本身。线程会不时地检测这个中断标志位，以判断线程是否应该被中断(中断标识是否位true)。它并不像stop()方法那样中断一个正在运行地线程  
interrupt()方法**只是改变中断状态，不会中断一个正在运行地线程**。需要用户自己去监视线程的状态。支持线程中断的方法(也就是线程中断后会抛出interruptedException的方法)就是在**监视线程的中断状态，一旦线程的中断状态被置为"中断状态"，就会抛出异常**。这一方法实际完成的是，给受阻塞的线程发出一个中断信号，这一受阻塞的线程检查到中断标识，就得以退出阻塞状态  
如果线程被Object.wait()、Thread.join()和Thread.sleep()三种方法之一阻塞，此时调用该线程的interrput()方法，那么该线程将抛出一个interruptedExection中断异常(该线程必须提前处理好此异常)，从而提早地终结被阻塞状态。如果线程没有被阻塞，这时调用interrupt(）将不起作用，直到执行到wait()、sleep()、join()时，才会马上抛出interruptedExection  
使用interrupt()方法不能真正的停止线程，仅仅是给线程打上了一个停止标记  
Thread提供了两个方法来判断  
```java
public static boolean interrupted()
```
```java
public boolean isInterrupted()
```
interrupted()方法是Thread类的静态方法，使用该方法不仅可以判断当前线程是否已经中断，而且还会清除该线程的标记  
isInterrputed()方法是Thread类的实例方法，使用该方法仅仅会判断当前线程是否已经中断，不会清除该线程的标记  
+ 使用interupterred()方法来退出线程  
```java
package com.sogou;


public class threadSeventh implements Runnable {

    @Override
    public void run() {
        try{
            for (int i = 0; i <= 5000; i++) {
                //判断当前线程是否已经中断
                if(Thread.interrupted()){
                    //会清除标记位，所以输出false
                    Thread.interrupted();
                    System.out.println(Thread.currentThread().isInterrupted());
                    //会清除标记位,所以输出false
                    Thread.interrupted();
                    System.out.println(Thread.currentThread().isInterrupted());
                    System.out.println("线程结束");
                    throw new InterruptedException();
                }
                System.out.println(i);
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        threadSeventh threadSeventh = new threadSeventh();
        Thread thread = new Thread(threadSeventh);
        thread.start();
        Thread.sleep(1);
        //线程中断，标记位设为true
        thread.interrupt();
    }
}
```
+ 使用isInterruped()方法来停止线程  
```java
package com.sogou;


public class threadSeventh implements Runnable {

    @Override
    public void run() {
        try{
            for (int i = 0; i <= 5000; i++) {
                //判断当前线程是否已经中断
                if(Thread.currentThread().isInterrupted()){
                    //不会清除标记位，所以输出位true
                    System.out.println(Thread.currentThread().isInterrupted());
                    System.out.println("线程结束");
                    throw new InterruptedException();
                }
                System.out.println(i);
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        threadSeventh threadSeventh = new threadSeventh();
        Thread thread = new Thread(threadSeventh);
        thread.start();
        Thread.sleep(1);
        thread.interrupt();
    }
}

```
##### interrupt()和sleep()方法的关系  

interrput()方法在java内部实际是设定一个标志位interrput status，可以中断任何阻塞状态，包括sleep()在内，当sleep()那行代码要阻塞的时候检查到这个标志位被设置为true，就会自己抛出异常InterrputedExection，并将这个标志位重置为false  
也就是说，如果先执行sleep()方法，后执行interrput()方法，阻塞会被打断，并抛出异常；如果先执行interrput()，后执行sleep()方法，则会直接抛出异常  
如下示例  
+ 先sleep()，后interrupt()  
```java
package com.sogou;

public class threadNingth implements  Runnable{

    @Override
    public void run() {
        System.out.println("开始线程");
        try {
            System.out.println(System.currentTimeMillis());
            Thread.sleep(100000);
            System.out.println(System.currentTimeMillis());
        }catch (InterruptedException e){
            System.out.println("线程中断");
            System.out.println(System.currentTimeMillis());
            e.printStackTrace();
        }
    }


    public static void main(String[] args) throws InterruptedException {
        threadNingth threadNingth = new threadNingth();
        Thread thread = new Thread(threadNingth);
        thread.start();
        Thread.sleep(5000);
        thread.interrupt();
    }
}
```
此时会立刻结束阻塞状态，并且抛出InterruptedExection的异常，这时就会结束线程  
+ 先执行interrupt(),后执行sleep()方法  
```java
package com.sogou;

public class threadNingth implements  Runnable{

    @Override
    public void run() {
        System.out.println("开始线程");
        for (int i = 0; i <50000 ; i++) {
            System.out.println(i);
        }
        System.out.println(Thread.currentThread().isInterrupted());
        try {
            System.out.println(System.currentTimeMillis());
            Thread.sleep(10000);
            System.out.println(System.currentTimeMillis());
        }catch (InterruptedException e){
            System.out.println("线程中断");
            System.out.println(System.currentTimeMillis());
            e.printStackTrace();
        }
    }


    public static void main(String[] args) throws InterruptedException {
        threadNingth threadNingth = new threadNingth();
        Thread thread = new Thread(threadNingth);
        thread.start();
        thread.interrupt();
    }
}
```
因为先执行interrupt()方法，所以先将interrupt status标志位设置为true，然后执行到sleep()方法的时候，检查该标志位位true，则会结束阻塞状态，抛出InterruptedExection异常，然后将interrupt status标志位重新置为false，结束线程  

**不过还是建议使用“抛出异常的方式”来实现线程的停止，因为在catch块种可以对异常信息进行相关的处理，而且使用异常能更好、更方便的控制程序的运行流程，不至于在代码中出现多个return，造成污染**  
#### yield方法  
yield()方法的作用是放弃当前的CPU资源，将它让给其他的任务去占用CPU时间。但放弃的时间不确定，有可能刚刚放弃，马上又获得CPU时间片  
```java
public static void yield()
//暂停当前正在执行的线程对象，并执行其他线程
```

```java
package com.sogou;

public class threadTenth implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i <500000 ; i++) {
            System.out.println(Thread.currentThread().getName()+"---"+i);
            //当前线程让出CPU使用权，进入就绪状态
            Thread.yield();
        }
    }

    public static void main(String[] args) {
        threadTenth threadTenth = new threadTenth();
        Thread thread1 = new Thread(threadTenth);
        Thread thread2 = new Thread(threadTenth);

        thread1.start();
        thread2.start();

    }
}
```
上述代码的结果不确定，就是因为当前线程放弃CPU使用权之后，在下一个线程执行的时候，此线程有可能被执行，也有可能不会被执行  
#### join  
线程的合并的含义就是**将几个并行线程合并为一个单线程**，应用场景是**当一个线程必须等待另一个线程执行完毕才能执行时**，Thread类提供给了join的实例方法来完成这个功能，注意**jion()方法不是静态方法**  
```java
void join()    
    当前线程等该加入该线程后面，等待该线程终止。    
void join(long millis)    
    当前线程等待该线程终止的时间最长为 millis 毫秒。 如果在millis时间内，该线程没有执行完，那么当前线程进入就绪状态，重新等待cpu调度   
void join(long millis,int nanos)    
    等待该线程终止的时间最长为 millis 毫秒 + nanos 纳秒。如果在millis时间内，该线程没有执行完，那么当前线程进入就绪状态，重新等待cpu调度
```
下面是一个例子  
```java
public class Test1 {  
    public static void main(String[] args) throws InterruptedException {  
        MyThread t=new MyThread();  
        t.start();  
        t.join(1);//将主线程加入到子线程后面，不过如果子线程在1毫秒时间内没执行完，则主线程便不再等待它执行完，进入就绪状态，等待cpu调度  
        for(int i=0;i<30;i++){  
            System.out.println(Thread.currentThread().getName() + "线程第" + i + "次执行！");  
        }  
    }  
}  
  
class MyThread extends Thread {  
    @Override  
    public void run() {  
        for (int i = 0; i < 1000; i++) {  
            System.out.println(this.getName() + "线程第" + i + "次执行！");  
        }  
    }  
}  
```
在JDK中，join方法的源码如下  
```java
public final synchronized void join(final long millis)
    throws InterruptedException {
        if (millis > 0) {
            if (isAlive()) {
                final long startTime = System.nanoTime();
                long delay = millis;
                do {
                    wait(delay);
                } while (isAlive() && (delay = millis -
                        TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - startTime)) > 0);
            }
        } else if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        } else {
            throw new IllegalArgumentException("timeout value is negative");
        }
    }
```
join方法实现是通过调用wait方法实现。当main线程调用t.join时候，main线程会获得线程对象的锁(wait意味着拿到该对象的锁)，调用该对象的wait(等待时间),直到该对象唤醒main线程，比如推出后，这就意味着main线程调用t.join时，必须能够拿到t对象的锁  
#### wait/notify  
##### wait()与notify()/notifyAll()方法必须在同步代码块中使用    
wait()、notify()/notifyAll()方法是Object类的方法，在执行两个方法时，必须要先获得锁，即在synchornized修饰的同步代码块或方法里调用wait()或notify()/notifyAll()方法  
##### wai()与notify()/notifyAll()的执行过程  
由于wait()与notify()/notifyAll()是放在同步代码块中的，**因此线程在执行它们时，肯定是进入了临界区中的，即该线程肯定是获得了锁的**  
当线程执行wait()方法时，会把当前的锁释放，然后让出CPU，进入等待状态  
当执行notify()/notifyAll()方法时，会唤醒一个处于等待该对象锁的线程，然后继续往下执行，直到执行完退出对象锁锁住的区域，后再释放锁  
可以看出，notify()/notify'All()方法执行后，并不会立即释放锁，而是需要等到执行完临界区中的代码后再释放。**应该再线程调用notify()/noyifyAll()方法后立即退出临界区，即不要在notify()/noyifyAll()后面在写一些耗时的代码**  

**synchornized wait notify/notifyAll三个的对象应该是用同一个对象**

```java
package com.sogou;

public class service {

    public void testWait(Object lock){
        try{
            synchronized (lock){
                System.out.println("begin wait"+ "   "+ Thread.currentThread().getName());
                lock.wait();
                System.out.println("end wait"+"    "+ Thread.currentThread().getName());
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }


    public void testNotify(Object lock){

        try{
            synchronized (lock){
                System.out.println("begin notify" + "    "+ Thread.currentThread().getName());
                System.out.println(System.currentTimeMillis());
                //唤醒被wait的进程
                lock.notify();
                //执行完notify之后不会立即释放锁，而是需要执行完同步代码块中红的内容后释放锁
                Thread.sleep(5000);
                System.out.println(System.currentTimeMillis());
                System.out.println("end notify"+"    "+Thread.currentThread().getName());
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}

class A implements Runnable{
    private Object lock;

    A(Object lock){
        this.lock=lock;
    }

    @Override
    public void run() {
        service service = new service();
        service.testWait(lock);
    }
}


class B implements Runnable{
    private Object lock;

    B(Object lock){
        this.lock=lock;
    }

    @Override
    public void run() {
        service service = new service();
        service.testNotify(lock);
    }
}


class test{

    public static void main(String[] args) throws InterruptedException {
        Object lock=new Object();
        A a = new A(lock);
        B b = new B(lock);
        Thread thread1 = new Thread(a);
        Thread thread2 = new Thread(b);
        thread1.start();
        Thread.sleep(1000);
        thread2.start();
    }

}
```
以上代码会在notify()之后再阻塞5秒钟才会释放锁，因此不建议再notify()/notifyAll()方法后面执行耗时的操作，而应该立即结束释放锁  
#### wait与interrupt  
wait也会像sleep方法一样检查当前线程的interrupt status的状态。如果标志位为true，那么wait方法不再被阻塞，会抛出一个InterrupedExection异常，但是并没有因此重新获得锁，所以能否向下执行还需要获得相应的锁  
如果先执行了wait()方法，后执行力interrupt()方法，调用wait()方法的线程不再被阻塞，而是抛出一个InterruptedExection异常，然后开始争抢锁  
如果先执行了interrupt()方法，然后执行了wait()方法，调用wait()方法会直接跳出，就像什么都没有发生一样，然后抛出一个InterruptedExection异常并开始争抢锁