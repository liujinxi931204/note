## 创建线程的方式  

一般来说创建线程有以下4钟方式  

### 继承Thread类  

 通过继承Thread类，并重写它的run方法，可以创建一个线程  

+ 首先定义一个类来继承Thread类，并重写run方法  

+ 然后创建这个子类对象，并调用start方法来启动线程  

```java
public class myThread extends Thread{
    @Override
    public void run(){
        System.out.println("thread run...");
    }
    
    public static void main(String[] args){
        new myThread().start();
    }
}
```

### 实现Runnable接口  

通过实现Runnable接口，并实现run方法，也可以创建一个线程  

+ 首先定义一个类实现Runnable接口，并实现run方法  
+ 然后创建Runnable实现类对象，并把它作为target传入到Thread的构造函数中  
+ 最后调用start方法开启线程

```java
public class myThread implements Runnable{
    @Override
    public void run(){
        System.out.println("thread run...");
    }
    
    public static void main(String[] args){
        new Thread(new myThread()).start();
    }
    
}  
```

### 实现Callable接口，并结合Future接口实现  

+ 首先定义一个Callable接口的实现类，并实现call方法。call方法带返回值  
+ 然后通过Future Task的构造函数，把这个Callable实现类传进去  
+ 把FutureTask作为Thread类的target，创建Thread线程对象  
+ 通过FutureTask的get方法获取线程的执行结果  

```java
public class TestFuture{
    public static void main(String[] args){
        FutureTask<Interget> task = new FutureTask<>(new myThread());
        new Thread(task).start();
        Interget result = task.get();//获取线程执行的结果
        System.out.println(result);
    }
}


public class myThread implements Callable<Interget>{
    @Override
    public Interget call() throws Exception{
        return new Random().nextInt(100);
    }
}
```

### 通过线程池创建线程  

这里使用JDK自带的Executors来创建线程池对象  

+ 首先定义一个Runnable的实现类，重写run方法  

+ 然后创建一个拥有固定线程数的线程池  

+ 最后通过ExecutorService对象的execute方法传入线程对象  

```java
public class myThread implements Runnable{
    
    @Override
    public void run(){
        System.out.println(Thread.currentThread().getName()+" thread run ...");
    }
    
    public static void main(String[] args){
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        for(int i=0;i<10;i++){
            executroService.execute(new myThread());
        }
        executorService.shutdown();
    }
}
```

看过Thread类的源码就知道，Thread类也是继承了Runnable接口的，所以创建线程主要就是实现Runnable接口和实现Callable接口两种方式，那么Runnable接口和Callable接口有什么区别呢？  

## Runnable接口和Callable接口  

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

可以发现，这个方法并没有返回值，如果我们希望执行某种类型的操作并且拿到它的返回值的时候，这个方法就不合适了  

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

可以发现这个方法有返回值并且还可以抛出异常  

所以对比Runnable接口和Callable接口我们可以发现  

+ Callable有返回值  

+ Callable可以抛出异常  

有返回值并不意外，因为这是我们的需求，call方法的返回值类型采用的是泛型，也就是在创建Callable对象的时候指定的  

抛出异常，意味着可以将异常抛出给任务的调用者来妥善处理，而run方法只能在方法内部捕获处理，这样就丧失了一定的灵活性  

那么如何获取call方法的返回值呢？可以使用下面的方式  

```java
public static void main(String[] args) {
    Callable<String> myCallable = () -> "This is the results.";
    try {
        String result = myCallable.call();
        System.out.println("Callable 执行的结果是: " + result);
    } catch (Exception e) {
        System.out.println("There is a exception.");
    }
}
```

这种方式确实可以获取到返回值，但是存在几个问题  

+ call方法是在当前线程中运行的，无法利用多线程  

+ 如果call方法是一个特别耗时的操作，那么该线程就必须等待，直到call方法返回  

+ 如果call方法始终不返回，那么就没有办法中断  

因此理想的方式是将call方法交给另一个线程去完成，并在合适的时候判断任务是否完成，然后获取结果或者撤销任务，这种思路就是Future接口  

## Future接口  

Future接口被设计用来代表一个异步操作的执行结果。可以用它来获取一个操作的执行结果、取消一个操作、判断一个操作是否已经完成或者是否被取消  

```java
public interface Future<V> {
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
    
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    
    boolean isDone();
}
```

Future接口一共定义了5个方法  

+ get()  

  该方法用来获取执行结果，如果任务还在执行中，就阻塞等待  

+ get(long timeout,TimeUnit unit)  

  该方法同get方法类似，所不同的是，它最多等待指定的时间。如果指定时间内任务没有完成，则会抛出TimeoutException  

+ cancel(boolean mayInterruptIfRunning)  

  该方法用来尝试取消一个任务的执行，它的返回值是boolean类型，表示取消操作是否成功  

+ isCancelled()

  该方法用于判断任务是否取消。如果一个任务在正常执行完成之前被cancel掉了，则返回true  

+ isDone()  

  如果一个任务已经结束，则返回true。注意，这里的任务结束包括了以下三种情况  

  + 任务正常执行完毕  

  + 任务抛出了异常  

  + 任务已经被取消  

关于cancel方法，有以下几点  

首先有以下三种情况之一的，cancel操作一定是失败的  

1. 任务已经被执行完了  

2. 任务已经被取消过了  

3. 任务因为某种原因不能被取消  

其他情况下，cancel操作将返回true。值得注意的是，cancel操作返回true并不代表任务真的就是被取消了，这取决于发动cancel状态时任务所处的状态  

1. 如果发起cancel时任务还没有开始执行，则随后的任务就不会被执行  

2. 如果发起cancel时任务已经在运行了，则这时就需要看mayInterruptRunning参数了  

   + 如果mayInterruptRunning为true，则当前在执行的任务会被中断  

   + 如果mayInterruptRunning为false，则可以允许正在执行的任务继续执行，直到它执行完  

## RunnableFuture接口  

RunnableFuture接口就是同时实现了Runnable接口和Future接口  

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run(); 
}
```

## FutureTask  

![img](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/1025005-20161030180319421-1644150953.png)  

FutureTask实现了RunnableFuture接口，也就是相当于它同时实现了Runnable接口和Future接口。Runnabel接口对应了Task，代表了FutureTask本质上也是表征了一个任务，而Future接口对应了Future，表示了对于这个任务可以执行某些操作，例如判断任务是否执行完毕，获取任务的执行结果，取消任务的执行等  

所以简单来说，FutureTask本质上是一个Task，可以把它当作Runnable对象来使用，但是它同时实现了Future接口，因此可以对它所代表的Task进行额外的控制操作  

java并发工具类有三个要点：状态、CAS、队列  

### 状态  

在FutureTask中，状态是由state属性来表示的，是volatile类型的，确保了不同线程对它修改的可见性  

```java
private volatile int state;
private static final int NEW          = 0;
private static final int COMPLETING   = 1;
private static final int NORMAL       = 2;
private static final int EXCEPTIONAL  = 3;
private static final int CANCELLED    = 4;
private static final int INTERRUPTING = 5;
private static final int INTERRUPTED  = 6;
```

state属性是贯穿整个FutureTask的最核心的属性，该属性的值代表了任务在运行过程中，随着任务的执行，状态将不断地进行转变，从上面地定义可以看出，总共有7种状态：包括1个初始状态，2个中间状态和4个终止状态  

但是状态的转换路径却只有四种  

![state of FutureTask](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/1460000016572594)  

+ 任务的初始状态都是NEW，这一点是由构造函数保证的  

+ 任务的终止状态有4种：

  NORMAL：表示任务已经完成，并且设置结果完成  

  EXCEPTIONAL：表示任务已经完成，并且设置异常完成  

  CANCELED：表示任务还没有开始执行就被取消了（非中断方式）  

  INTERRUPTED：表示任务还没有开始执行就被取消了（中断方式），且已经被中断  

+ 任务中间中间状态  

  COMPLETING：任务已经执行完毕，正在设置任务结果（正常结束或者异常结束）

  INTERRUPTING：任务还没有完成被中断，正在中断运行任务的线程  

值得一提的是，任务的中间状态是一个瞬态，非常短暂。而且任务的中间状态不代表任务正在执行，而是任务已经执行完了，而是任务已经执行完了，正在设置最终的返回结果，所以可以这么说，**只要state不处于NEW状态，就说明任务已经执行完毕**  

而将一个任务的状态设置成终止状态只有三种方法  

+ set  

+ setException  

+ cancel  

各个状态之间的转换  

![clipboard.png](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/bVbiwLO)  

### 队列  

在FutureTask中，队列的实现是一个单向链表，它表示所有等待任务执行完毕的线程的集合。我们知道FutureTask实现了Future接口，可以获取Task的执行结果。那么获取结果时任务还没有执行完毕怎么办呢？就将获取结果的线程包装成一个节点，然后在等待队列中挂起，直到任务执行完毕被唤醒。  

来看一看每个节点的结构是怎么样的  

```java
static final class WaitNode {
    volatile Thread thread;
    volatile WaitNode next;
    WaitNode() { thread = Thread.currentThread(); }
}
```

可见，相比较于AQS的等待队列所使用的双向链表的Node，这个WaitNode仅仅包含一个记录线程的Thread属性和指向下一个节点的next属性  

值得一提的是，FutureTask中的这个单向链表是当作栈来使用的，确切来说是当作Treiber栈来使用的，可以当作是一个线程安全的栈，它使用CAS操作完成入栈出栈操作。这是因为同一时刻可能存在多个线程都在获取任务的执行结果，如果任务还在执行过程中，则者这些线程就要被包装成WaitNode节点加入到栈顶，因此需要CAS操作保证入栈、出栈操作的线程安全  

由于FutureTask中的队列本质是一个栈，那么使用这个栈就需要一个指向栈顶节点的指针，在FutureTask中就是waiters属性  

```java
/** Treiber stack of waiting threads */
private volatile WaitNode waiters;
```

FutureTask中所使用的队列如下所示  

![Treiber stack](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/1460000016572595)  

### 核心属性  

```java
    /**
     * The run state of this task, initially NEW.  The run state
     * transitions to a terminal state only in methods set,
     * setException, and cancel.  During completion, state may take on
     * transient values of COMPLETING (while outcome is being set) or
     * INTERRUPTING (only while interrupting the runner to satisfy a
     * cancel(true)). Transitions from these intermediate to final
     * states use cheaper ordered/lazy writes because values are unique
     * and cannot be further modified.
     *
     * Possible state transitions:
     * NEW -> COMPLETING -> NORMAL
     * NEW -> COMPLETING -> EXCEPTIONAL
     * NEW -> CANCELLED
     * NEW -> INTERRUPTING -> INTERRUPTED
     */
    private volatile int state;
    private static final int NEW          = 0;
    private static final int COMPLETING   = 1;
    private static final int NORMAL       = 2;
    private static final int EXCEPTIONAL  = 3;
    private static final int CANCELLED    = 4;
    private static final int INTERRUPTING = 5;
    private static final int INTERRUPTED  = 6;

    /** The underlying callable; nulled out after running */
    private Callable<V> callable;
    /** The result to return or exception to throw from get() */
    private Object outcome; // non-volatile, protected by state reads/writes
    /** The thread running the callable; CASed during run() */
    private volatile Thread runner;
    /** Treiber stack of waiting threads */
    private volatile WaitNode waiters;
```

可以看出，FutureTask的核心属性有5个  

+ state：当前任务的状态

+ callable：代表了要执行的任务本身，即FutureTask中的Task

+ outcome：代表了任务的执行结果或者抛出的异常，为Object类型

+ runner：任务的执行者，即执行Task的线程

+ waiters：指向队列中的头节点或者说是指向栈中的栈顶元素

### 构造函数 

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}
```

```java
public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       // ensure visibility of callable
}
```

FutureTask一共有两个构造函数，一个是传入一个Callable对象，一个是传入一个Runnable对象和一个指定的返回类型，然后通过Executors工具类将它是配成一个callable对象，所以这两个构造函数的本质是一样的  

+ 同传入的参数初始化callable属性

+ 将FutureTask的状态设置为NEW  

FutureTask实现了RunnableFuture接口，那么就必须实现Runnable和Future接口的所有方法，先来看一下Runnable接口的run方法  

### run方法  

```java
public void run() {
    //如果当前状态不是NEW或者cas操作设置runner为当前线程失败，记录执行任务的线程失败，直接返回
    if (state != NEW || !UNSAFE.compareAndSwapObject(this, runnerOffset, null, Thread.currentThread()))
        return;
    try {
        //任务本身
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                set(result);
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```





