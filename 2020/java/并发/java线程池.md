## 线程池是什么  

线程池是一种基于池化思想管理线程的工具，经常出现在多线程服务中  

### 不使用线程所带来的问题  

反复创建销毁线程会带来处理任务以外的额外开销。如果需要执行的任务比较简单，那么有可能造成创建销毁线程的所占用的资源超过了执行任务所占用的资源  

如果当任务过多时，每个线程负责一个任务，那么需要创建很多线程区执行任务，过多的线程会导致占用大量的资源和线程的上下文开销，同时还会导致系统的不稳定

### 使用线程池的好处  

+ 减低资源消耗：通过池化技术重复利用线程池中的线程，可以减少创建销毁线程所带来的开销  

+ 提高响应速度：任务到达时可以立即执行

+ 提高线程的客观理性：线程是稀缺资源，如果无限制的创建不仅会消耗系统的资源，还会降低系统的稳定性

+ 提供更强大的功能：线程池具备可扩展性，允许开发人员向其中增加更多的功能  

## Executors框架  

### 从Executor谈起  

Executor是jdk 1.5时，随着J.U.C引入的一个接口，引入该接口的目的就是解耦任务的本身和任务的执行。之前执行一个任务时，往往需要先创建一个线程，然后调用这个线程的start方法来执行任务  

```java
new Thread(new(RunnableTask())).start();
```

上述RunnableTask是实现了Runnable接口的任务类  

而executor接口解耦了任务本身和任务的执行，该接口只有一个方法，入参是需要执行的任务  

```java
public interface Executor {
    /**
     * 执行给定的Runnable任务.
     * 根据Executor的实现不同, 具体执行方式也不相同.
     *
     * @param command the runnable task
     * @throws RejectedExecutionException if this task cannot be accepted for execution
     * @throws NullPointerException       if command is null
     */
    void execute(Runnable command);
}
```

这时候就可以像下面这种执行任务，而不用关心线程的创建了  

```java
Executor executor = someExecutor;       // 创建具体的Executor对象
executor.execute(new RunnableTask1());
executor.execute(new RunnableTask2());
...
```

由于executor只是一个接口，所以根据其实现不同，执行任务的具体方式也不同  

### 增强的Executor——ExecutorService  

Executor接口提供的功能比较简单，为了对它的功能进行增强，J.U.C又提供了另一个接口ExecutorService，

```java
public interface ExecutorService extends Executor
```

可以看到，ExecutorService接口继承了Executor接口，它在Executor的基础上，增强了对任务的控制，以及对自身生命周期的管理，主要有四类

1. 关闭执行器，禁止任务提交  
2. 监视执行器的状态
3. 提供对异步任务的支持
4. 提供对批处理任务的支持

```java
public interface ExecutorService extends Executor {

    /**
     * 关闭执行器, 主要有以下特点:
     * 1. 已经提交给该执行器的任务将会继续执行, 但是不再接受新任务的提交;
     * 2. 如果执行器已经关闭了, 则再次调用没有副作用.
     */
    void shutdown();

    /**
     * 立即关闭执行器, 主要有以下特点:
     * 1. 尝试停止所有正在执行的任务, 无法保证能够停止成功, 但会尽力尝试(例如, 通过 Thread.interrupt中断任务, 但是不响应中断的任务可能无法终止);
     * 2. 暂停处理已经提交但未执行的任务;
     *
     * @return 返回已经提交但未执行的任务列表
     */
    List<Runnable> shutdownNow();

    /**
     * 如果该执行器已经关闭, 则返回true.
     */
    boolean isShutdown();

    /**
     * 判断执行器是否已经【终止】.
     * <p>
     * 仅当执行器已关闭且所有任务都已经执行完成, 才返回true.
     * 注意: 除非首先调用 shutdown 或 shutdownNow, 否则该方法永远返回false.
     */
    boolean isTerminated();

    /**
     * 阻塞调用线程, 等待执行器到达【终止】状态.
     *
     * @return {@code true} 如果执行器最终到达终止状态, 则返回true; 否则返回false
     * @throws InterruptedException if interrupted while waiting
     */
    boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;

    /**
     * 提交一个具有返回值的任务用于执行.
     * 注意: Future的get方法在成功完成时将会返回task的返回值.
     *
     * @param task 待提交的任务
     * @param <T>  任务的返回值类型
     * @return 返回该任务的Future对象
     * @throws RejectedExecutionException 如果任务无法安排执行
     * @throws NullPointerException       if the task is null
     */
    <T> Future<T> submit(Callable<T> task);

    /**
     * 提交一个 Runnable 任务用于执行.
     * 注意: Future的get方法在成功完成时将会返回给定的结果(入参时指定).
     *
     * @param task   待提交的任务
     * @param result 返回的结果
     * @param <T>    返回的结果类型
     * @return 返回该任务的Future对象
     * @throws RejectedExecutionException 如果任务无法安排执行
     * @throws NullPointerException       if the task is null
     */
    <T> Future<T> submit(Runnable task, T result);

    /**
     * 提交一个 Runnable 任务用于执行.
     * 注意: Future的get方法在成功完成时将会返回null.
     *
     * @param task 待提交的任务
     * @return 返回该任务的Future对象
     * @throws RejectedExecutionException 如果任务无法安排执行
     * @throws NullPointerException       if the task is null
     */
    Future<?> submit(Runnable task);

    /**
     * 执行给定集合中的所有任务, 当所有任务都执行完成后, 返回保持任务状态和结果的 Future 列表.
     * <p>
     * 注意: 该方法为同步方法. 返回列表中的所有元素的Future.isDone() 为 true.
     *
     * @param tasks 任务集合
     * @param <T>   任务的返回结果类型
     * @return 任务的Future对象列表，列表顺序与集合中的迭代器所生成的顺序相同，
     * @throws InterruptedException       如果等待时发生中断, 会将所有未完成的任务取消.
     * @throws NullPointerException       任一任务为 null
     * @throws RejectedExecutionException 如果任一任务无法安排执行
     */
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;

    /**
     * 执行给定集合中的所有任务, 当所有任务都执行完成后或超时期满时（无论哪个首先发生）, 返回保持任务状态和结果的 Future 列表.
     */
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException;

    /**
     * 执行给定集合中的任务, 只有其中某个任务率先成功完成（未抛出异常）, 则返回其结果.
     * 一旦正常或异常返回后, 则取消尚未完成的任务.
     */
    <T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;

    /**
     * 执行给定集合中的任务, 如果在给定的超时期满前, 某个任务已成功完成（未抛出异常）, 则返回其结果.
     * 一旦正常或异常返回后, 则取消尚未完成的任务.
     */
    <T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

### 周期任务的调度——ScheduledExecutorService

在ExecutorService的基础上，J.U.C又提供了一个接口ScheduledExecutorService，该接口主要是为了满足某些任务能够定时执行或者周期性的执行

```java
public interface ScheduledExecutorService extends ExecutorService
```

ScheduledExecutorService提供了一系列schedule方法，可以在给定的延迟后执行提交的任务，或者每隔指定的周期执行一次任务

public interface ScheduledExecutorService extends ExecutorService {

```java
/**
 * 提交一个待执行的任务, 并在给定的延迟后执行该任务.
 *
 * @param command 待执行的任务
 * @param delay   延迟时间
 * @param unit    延迟时间的单位
 */
public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit);
 
/**
 * 提交一个待执行的任务（具有返回值）, 并在给定的延迟后执行该任务.
 *
 * @param command 待执行的任务
 * @param delay   延迟时间
 * @param unit    延迟时间的单位
 * @param <V>     返回值类型
 */
public <V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit);
 
/**
 * 提交一个待执行的任务.
 * 该任务在 initialDelay 后开始执行, 然后在 initialDelay+period 后执行, 接着在 initialDelay + 2 * period 后执行, 依此类推.
 *
 * @param command      待执行的任务
 * @param initialDelay 首次执行的延迟时间
 * @param period       连续执行之间的周期
 * @param unit         延迟时间的单位
 */
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit);
 
/**
 * 提交一个待执行的任务.
 * 该任务在 initialDelay 后开始执行, 随后在每一次执行终止和下一次执行开始之间都存在给定的延迟.
 * 如果任务的任一执行遇到异常, 就会取消后续执行. 否则, 只能通过执行程序的取消或终止方法来终止该任务.
 *
 * @param command      待执行的任务
 * @param initialDelay 首次执行的延迟时间
 * @param delay        一次执行终止和下一次执行开始之间的延迟
 * @param unit         延迟时间的单位
 */
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit);
```
以上就是Executor框架中三个最核心的接口，这三个接口的关系以及其中主要的方法如下图

![img](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/3724012404-5bbec224900b5_fix732.png)  

## 生产Executor的工厂  

通过上一部分，我们知道Executors框架就是同来解耦任务本身与任务的执行的，并提供了三个核心的方法来满足开发人员的需求：

- Executor：提交普通的可执行任务
- ExecutorService：提供对线程池生命周期的管理、异步任务的支持
- ScheduledExecutorService：提供对任务的周期性支持

同时J.U.C还提供了一个Executors类，专门用于创建上述接口的实现类对象。Executors其实就是一个简单工厂，它的所有方法都是静态的，用户可以根据需要，选择需要创建的执行器实例，Executors一共提供了五类Executor执行器实例

### 固定线程数量的线程池

Executors提供了下面的两种方法创建具有固定线程数量的Executor的方法，固定线程池在初始化时确定其中的线程总数，运行过程中保持不变

```java
/**
 * 创建一个具有固定线程数的Executor.
 */
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS,
            new LinkedBlockingQueue<Runnable>());
}

/**
 * 创建一个具有固定线程数的Executor.
 * 在需要时使用提供的 ThreadFactory 创建新线程.
 */
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS,
            new LinkedBlockingQueue<Runnable>(), threadFactory);

}
```

### 单个线程的线程池

Executors提供了两种方法来创建只有单个线程的线程池

```java
/**
 * 创建一个使用单个 worker 线程的 Executor.
 */
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS,
                    new LinkedBlockingQueue<Runnable>()));
}
 
/**
 * 创建一个使用单个 worker 线程的 Executor.
 * 在需要时使用提供的 ThreadFactory 创建新线程.
 */
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
    return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS,
                    new LinkedBlockingQueue<Runnable>(), threadFactory));
}
```

### 可缓存的线程池

有些情况下，我们虽然创建了具有一定线程数的线程池，但出于对资源利用率的考虑，可能希望在特定的时候对线程进行回收，这时候就需要使用可缓存的线程池了

```java
/**
 * 创建一个可缓存线程的Execotor.
 * 如果线程池中没有线程可用, 则创建一个新线程并添加到池中;
 * 如果有线程长时间未被使用(默认60s, 可通过threadFactory配置), 则从缓存中移除.
 */
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS,
            new SynchronousQueue<Runnable>());
}
 
/**
 * 创建一个可缓存线程的Execotor.
 * 如果线程池中没有线程可用, 则创建一个新线程并添加到池中;
 * 如果有线程长时间未被使用(默认60s, 可通过threadFactory配置), 则从缓存中移除.
 * 在需要时使用提供的 ThreadFactory 创建新线程.
 */
public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS,
            new SynchronousQueue<Runnable>(), threadFactory);
}
```

### 可延时/周期调度的线程池

如果有任务需要延迟/周期调用，就需要这种线程池

```java
/**
 * 创建一个具有固定线程数的 可调度Executor.
 * 它可安排任务在指定延迟后或周期性地执行.
 */
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
 
/**
 * 创建一个具有固定线程数的 可调度Executor.
 * 它可安排任务在指定延迟后或周期性地执行.
 * 在需要时使用提供的 ThreadFactory 创建新线程.
 */
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize, ThreadFactory threadFactory) {
    return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
}
```

### Fork/Join线程池

Fork/Join线程池是一类比较特殊的线程池，其核心就是ForkJoinPool，主要面对的是Fork/Join框架

```java
/**
 * 创建具有指定并行级别的ForkJoin线程池.
 */
public static ExecutorService newWorkStealingPool(int parallelism) {
    return new ForkJoinPool(parallelism, ForkJoinPool.defaultForkJoinWorkerThreadFactory, null, true);
}
 
/**
 * 创建并行级别等于CPU核心数的ForkJoin线程池.
 */
public static ExecutorService newWorkStealingPool() {
    return new ForkJoinPool(Runtime.getRuntime().availableProcessors(), ForkJoinPool.defaultForkJoinWorkerThreadFactory,
            null, true);
}
```

## 总结

整个Executors框架基本结构就如下图

![img](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/Executors%E6%A1%86%E6%9E%B6.png)    

## ThreadPoolExecutor简介

ThreadPoolExecutor，是J.U.C提供的一种实现了ExecutorService接口的执行器，也就是线程池。但是ThreadPoolExecutor没有直接实现ExecutorService接口，因为它只是ExecutorService接口的一种实现而已，所以Doug Lea把一些通用的部分封装成一个抽象父类AbstractExecutorService，供J.U.C中的其他执行器继承。如果需要实现一个Executor，也可以继承这个类

```java
public class ThreadPoolExecutor extends AbstractExecutorService
```

### AbstractExecutorService

AbstractExecutorService主要实现了ExecutorService中submit、invokeAny、invokeAll这三类方法。这三类方法的返回值几乎都是一个Future对象。而Future是一个接口，AbstractExecutorService既然实现了这些方法，必然实现了这个Future接口，来看一下AbstractExecutorService的submit方法

```java
public <T> Future<T> submit(Runnable task, T result) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task, result);
    execute(ftask);
    return ftask;
}
```

可以看到，上述方法首先对Runnable任务和返回值value进行了封装，通过一个newTaskFor的方法，封装成了一个FutureTask对象，然后通过execute方法执行任务，最后返回异步任务对象

这里其实是模板方法的运用，execute方法是一个抽象方法，需要由继承了AbstractExecutorService子类实现具体的逻辑

来看一下newTaskFor方法

```java
protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
    return new FutureTask<T>(runnable, value);
}
```

[FutureTask](https://gitee.com/liujinxi931204/note/blob/master/2020/java/并发/FutureTask.md)其实就是Future接口的实现类

![img](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/FutureTask%E7%BB%A7%E6%89%BF%E7%BB%93%E6%9E%84.png)  

### 线程池简介

当有任务需要执行时，线程池会给该任务分配线程，如果当前没有可用线程，一般会将任务放进一个队列中，当有可用的线程时，再从队列中取出任务并执行

![img](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E7%BA%BF%E7%A8%8B%E6%B1%A0%E7%AE%80%E4%BB%8B.png)  

## ThreadPoolExecutor基本原理

### 构造线程池

Executors的工厂方法可以创建三种线程池：newFixedThreadPool，newSingleThreadPool，newCachedThreadPool。但其实创建它们的方法内部都调用了ThreadPoolExecutor的构造方法来实例化ThreadPoolExecutor对象

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

这个构造方法一共有7个参数

- corePoolSize：核心线程池中的最大线程数

- maximumPoolSize：总线成中的最大线程数

- keepAliveTime：空闲线程的存活时间

- unit：keepAliveTime的时间单位

- workQueue：任务队列，用来保存已经提交但是还没有执行的任务

- threadFactory：线程工厂（用于指定如果创建一个线程）

- handler：拒绝策略（当任务太多导致工作队列满时的处理策略）

正是通过上述参数的组合，Executors工厂可以创建不同类型的线程池，这里简单说一下corePoolSize和maximumPoolSize这两个参数

#### 核心线程池和非核心线程池

ThreadPoolExecutor在逻辑上将自身管理的线程池分为核心线程池（大小对应corePoolSize）和非核心线程池（大小对应maximumPoolSize-corePoolSize）

当我们向线程池提交一个任务时，将创建一个工作线程，称之为Worker，Worker在逻辑上属于核心线程池还是非核心线程池，要根据corePoolSize、maximumPoolSize和Worker的总数进行判断

![img](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E6%80%BB%E7%BA%BF%E7%A8%8B%E6%B1%A0.png)  

ThreadPoolExecutor中只有一种类型的线程，名叫Worker，它是ThreadPoolExecutor定义的内部类，同时封装着Runnable任务和执行该任务的Thread对象，它也是ThreadPoolExecutor唯一需要进行维护的线程；核心线程池和非核心线程池则是逻辑上的概念。

核心线程池和非核心线程池的使用如下（可以通过下面对线程池的调度流程分析印证）

1. 如果工作线程数小于核心线程池的上限，则直接创建一个工作线程执行任务

2. 如果工作线程数大于核心线程池的上限，但是任务队列没有满，则将任务加入到任务队列等待

3. 如果工作线程数大于核心线程池的上限，但是任务队列已满，这时就需要对比总线程池的数量了

   3.1 如果工作线程数大于核心线程池的上限，且又小于总线程池的上限，则在非核心线程池中创建线程执行任务

   3.2 如果工作线程数大于核心线程池的上限，且又大于总线程池的上限，则执行拒绝策略

核心线程池：固定线程数，可闲置，默认不会被销毁，如果设置allowCoreThreadTimeOut属性为true时，keepAliveTime会作用于核心线程，如果线程闲置的时间超过这个时长，线程会被回收

非核心线程池：如果线程闲置的时长超过了keepAliveTime，线程会被回收

### 工作线程  

工作线程Worker被定义为ThreadPoolExecutor的一个内部类，实现了AQS框架，ThreadPoolExecutor通过一个HashSet来保存工作线程  

```java
/**
 * 工作线程集合.
 */
private final HashSet<Worker> workers = new HashSet<Worker>();
```

工作线程的定义如下：

```java
/**
 * Worker表示线程池中的一个工作线程, 可以与任务相关联.
 * 由于实现了AQS框架, 其同步状态值的定义如下:
 * -1: 初始状态
 * 0:  无锁状态
 * 1:  加锁状态
 */
private final class Worker extends AbstractQueuedSynchronizer implements Runnable {
 
    /**
     * 与该Worker关联的线程.
     */
    final Thread thread;
    /**
     * Initial task to run.  Possibly null.
     */
    Runnable firstTask;
    /**
     * Per-thread task counter
     */
    volatile long completedTasks;
 
 
    Worker(Runnable firstTask) {
        setState(-1); // 初始的同步状态值
        this.firstTask = firstTask;
        this.thread = getThreadFactory().newThread(this);
    }
 
    /**
     * 执行任务
     */
    public void run() {
        runWorker(this);
    }
 
    /**
     * 是否加锁
     */
    protected boolean isHeldExclusively() {
        return getState() != 0;
    }
 
    /**
     * 尝试获取锁
     */
    protected boolean tryAcquire(int unused) {
        if (compareAndSetState(0, 1)) {
            setExclusiveOwnerThread(Thread.currentThread());
            return true;
        }
        return false;
    }
 
    /**
     * 尝试释放锁
     */
    protected boolean tryRelease(int unused) {
        setExclusiveOwnerThread(null);
        setState(0);
        return true;
    }
 
    public void lock() {
        acquire(1);
    }
 
    public boolean tryLock() {
        return tryAcquire(1);
    }
 
    public void unlock() {
        release(1);
    }
 
    public boolean isLocked() {
        return isHeldExclusively();
    }
 
    /**
     * 中断线程(仅任务非初始状态)
     */
    void interruptIfStarted() {
        Thread t;
        if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
            try {
                t.interrupt();
            } catch (SecurityException ignore) {
            }
        }
    }
}
```

通过Worker的定义可以看到，每个Worker对象都一个Thread线程对象与它相对应，当任务需要执行时，实际调用内部Thread对象的start方法，而Thread对象是在Worker的构造器中通过getThreadFactory().newThread(this)方法创建，创建的Thread对象将Worker自身作为任务，所以调用Thread的start方法时，实际最终调用了Worker.run()方法，该方法内部委托给runWorker方法执行任务  

### 线程工厂  

ThreadFactory用来创建单个线程，当线程池需要创建一个线程时，就调用该类的newThread(Runnable r)方法创建线程（ThreadPoolExecutor中实际创建线程的时刻是将任务包装成工作线程Worker的时候）

ThreadPoolExecutor在构造时如果不指定ThreadFactory，则默认使用Executors.defaultThreadFactory()来创建一个ThreadFactory  

```java
public static ThreadFactory defaultThreadFactory() {
    return new DefaultThreadFactory();
}

/**
 * 默认的线程工厂.
 */
static class DefaultThreadFactory implements ThreadFactory {
    private static final AtomicInteger poolNumber = new AtomicInteger(1);
    private final ThreadGroup group;
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;
 
    DefaultThreadFactory() {
        SecurityManager s = System.getSecurityManager();
        group = (s != null) ? s.getThreadGroup() : Thread.currentThread().getThreadGroup();
        namePrefix = "pool-" + poolNumber.getAndIncrement() + "-thread-";
    }
 
    public Thread newThread(Runnable r) {
        Thread t = new Thread(group, r, namePrefix + threadNumber.getAndIncrement(), 0);
        if (t.isDaemon())
            t.setDaemon(false);
        if (t.getPriority() != Thread.NORM_PRIORITY)
            t.setPriority(Thread.NORM_PRIORITY);
        return t;
    }
}
```

使用线程工厂的好处是，一来可以解耦对象的创建和使用，二来是可以批量配置线程信息（包括优先级、线程名称、是否是守护线程），以自由设置池子中所有线程的状态

### 线程池状态和线程管理  

ThreadPoolExecutor内部定义了一个AtomicInteger变量——ctl，通过按位划分的方式，在一个变量中记录线程池状态和工作线程数——低29位保存线程数，高3位保存线程状态

```java
/**
 * 保存线程池状态和工作线程数:
 * 低29位: 工作线程数
 * 高3位 : 线程池状态
 */
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
 
private static final int COUNT_BITS = Integer.SIZE - 3;
 
// 最大线程数: 2^29-1
private static final int CAPACITY = (1 << COUNT_BITS) - 1;  // 00011111 11111111 11111111 11111111
 
// 线程池状态
private static final int RUNNING = -1 << COUNT_BITS;        // 11100000 00000000 00000000 00000000
private static final int SHUTDOWN = 0 << COUNT_BITS;        // 00000000 00000000 00000000 00000000
private static final int STOP = 1 << COUNT_BITS;            // 00100000 00000000 00000000 00000000
private static final int TIDYING = 2 << COUNT_BITS;         // 01000000 00000000 00000000 00000000
private static final int TERMINATED = 3 << COUNT_BITS;      // 01100000 00000000 00000000 00000000
```

可以看到，ThreadPoolExecutor一共定义了5种线程池状态  

+ RUNNING：接受新任务，且处理已经进入阻塞队列的任务
+ SHUTDOWN：不接受新任务，但处理已经进入阻塞队列的任务
+ STOP：不接受新任务，且不处理已经进入阻塞队列的任务，同时中断正在执行的任务
+ TIDYING：所有任务都已经终止，工作线程数为0，线程转化为TIDYING并准备调用terminated方法
+ TERMINATED：terminated方法已经被调用

![线程池状态流转](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E7%BA%BF%E7%A8%8B%E6%B1%A0%E7%8A%B6%E6%80%81%E6%B5%81%E8%BD%AC.png)  

## 线程池的调度流程

ThreadPoolExecutor没有重写submit方法，而是沿用了父类AbstractExecutorService中的submit方法，然后自己实现了execute方法的逻辑  

```java
public Future<?> submit(Runnable task) {
    //提交的任务不能为空
    if (task == null) throw new NullPointerException();
    //将runnable任务转换为FutureTask实例
    RunnableFuture<Void> ftask = newTaskFor(task, null);
    //execute的具体逻辑由子类自己实现
    execute(ftask);
    return ftask;
}
```

再来看ThreadPoolExecutor具体实现execute方法  

```java
public void execute(Runnable command) {
    //执行的任务不能为空
    if (command == null)
        throw new NullPointerException();
    //获取当前的ctl，ctl的低29位保存的是线程的数量
    int c = ctl.get();
    //workerCountOf获取当前工作线程的数量
    //工作线程数<corePoolSize上限，尝试添加工作线程并执行
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        //并发环境中，有可能获取当前工作线程数量的时候可以执行，但是添加的时候存在别的线程已经先添加了的情况，导致添加，导致工作线程失败，获取新的ctl
        c = ctl.get();
    }
    //执行到这里说明工作线程数>corePoolSize上限，或者上面所述添加工作线程失败，也说明工作线程数>corePoolSize上限
    //如果线程池是运行状态，向工作队列中添加任务
    
    /**
    * isRunning方法返回当前是否小于0
    * private static boolean isRunning(int c) {
    *    return c < SHUTDOWN;
    * }
    */
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        //重新检查线程池的状态，如果线程池不是running状态，删除上一步添加的任务
        if (! isRunning(recheck) && remove(command))
            //执行拒绝策略
            reject(command);
        //到这里说明
        //1. 线程池处于running状态
        //2. 线程池没有处于running状态，但是删除任务失败
        
        //workerCountOf(recheck) == 0对应一个特殊情况，就是线程池已处于shutdown状态，但是工作队列中还有任务没有执行完，需要新创建一个线程将任务执行完
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
   //1. 说明线程池没有处于running的状态
   //2. 线程池处于running的状态，但是workQueue.offer(command)失败，说明工作队列已满，需要在非核心线程池创建线程执行任务
    else if (!addWorker(command, false))
        //添加线程失败，说明工作线程数>maximumPoolSize,执行拒绝策略
        reject(command);
}
```

通过execute方法可以印证上面我们对于核心线程池、非核心线程池以及拒绝策略的流程分析。

1. 如果工作线程数小于核心线程池的上限，则直接创建一个工作线程执行任务

2. 如果工作线程数大于核心线程池的上限，但是任务队列没有满，则将任务加入到任务队列等待

3. 如果工作线程数大于核心线程池的上限，但是任务队列已满，这时就需要对比总线程池的数量了

   3.1 如果工作线程数大于核心线程池的上限，且又小于总线程池的上限，则在非核心线程池中创建线程执行任务

   3.2 如果工作线程数大于核心线程池的上限，且又大于总线程池的上限，则执行拒绝策略

有一点需要注意，就是线程池处于shutdown的状态时，没有活动的线程了，但是工作队列中还有没有执行完的任务，这时线程池会创建一个线程来执行这些在工作队列中的任务。

上述execute方法可以用如下流程图来表示  

![线程池调度流程](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E7%BA%BF%E7%A8%8B%E6%B1%A0%E8%B0%83%E5%BA%A6%E6%B5%81%E7%A8%8B.png)  

### 工作线程的创建  

在上面的execute方法中，会调用addWorker方法来创建工作线程并执行，现在就来看一下这个方法  

```java
/**
* firstTask，如果指定了参数，表示立即创建一个线程执行这个任务，如果为空；复用已有的工作线程，执行队列中的任务
* core，ture表示线程属于核心线程池；false表示线程属于非核心线程池
*/
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        //获取当前的ctl
        int c = ctl.get();
        //获取当前的运行状态
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        /**
        * 线程池的状态为非running && （线程池状态 != shutdown || firstTask != null || 队列为空）
        * 有以下三种情况
        * 1. 线程池的状态为非running && 线程池的状态为非shutdown
        * 即线程池的状态为stop、tidying、terminated三者中的任意一个，线程池不再接受新任务，所以通过addWorker新添加任务会失败
        * 2. 线程池的状态为非running && firstTask不为空
        * 即线程池的状态为shutdown、stop、tidying、terminated并且firstTask不为空，此时线程池不再接受新任务，所以addWorker添加新任务会失败，但会处理队列中的任务
        * 3. 线程池状态为非running && 队列不为空
        * 即线程池的状态为shutdown、stop、tidying、terminated并且队列为空，此时线程池不再接受新任务，并且队列中也没有需要处理的任务
        即
        */
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        //线程池的状态为running 或者 线程池的状态为shutdown并且工作队列中还有任务没有执行完
        for (;;) {
            //获取当前工作线程数量
            int wc = workerCountOf(c);
            //如果当前工作线程数超过了允许的最大值
            //或者对核心线程池，工作线程数超过了核心线程池数量的上线
            //或者总的线程数超过了总的最大线程数，都返回false
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            //cas操作，将工作线程数加一
            if (compareAndIncrementWorkerCount(c))
                //添加成功跳出循环
                break retry;
            //重新获取ctl，如果线程池的状态发生了改变，需要重新进入循环，进行重试，其实就是自旋
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }

    //以下是添加worker的过程
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        //新创建一个worker对象，方法内会使用线程工厂新创建一个线程，去执行任务
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                //重新检查线程池的状态，有可能在获取锁的前一步，线程池已经被终止了
                int rs = runStateOf(ctl.get());

                //线程池状态为running或者线程池为shutdown 并且队列中还有任务需要执行
                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    //加入工作线程集合，workers是一个HashSet
                    workers.add(w);
                    int s = workers.size();
                    //维护一个全局达到过的最大线程数计数器
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            //worker添加成功后，将worker启动起来执行任务
            if (workerAdded) {
                //会执行worker的run方法，而在run方法内部又会调用worker的runWorker方法
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        //获取锁的前一步，线程池已经被终止了，导致workerAdded为false
        if (! workerStarted)
            addWorkerFailed(w);
    }
    //返回状态，表示添加成功或者失败
    return workerStarted;
}
```

可以看到addWorker分为两个只要的部分

第一部分主要是一个自旋的操作，主要是对线程池的状态进行一些判断，如果状态不合适接受新任务，或者工作线程数超出了限制，则直接返回false  

这里需要注意参数core，如果core为true，表示新建的工作线程逻辑上属于核心线程池，所以需要判断工作线程数>corePoolSize；如果core为false，表示新建的工作线程逻辑上属于非核心线程池，所以需要判断工作线程数>maximumPoolSize  

第二部分主要是经过第一个过滤以后，去真正的创建工作线程并执行任务，首先将Runnable任务包装成worker对象，然后将worker加入到一个工作线程集合中，最后调用worker的run方法去执行任务

### 工作线程的执行  

上面的submit方法其实包含有工作线程的创建、执行、销毁以及执行拒绝策略。这里现来看一下工作线程的执行，这个方法在worker的run方法中

```java
public void run() {
    runWorker(this);
}
```

可以看到run方法内部又调用了worker的runWorker方法  

```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    //worker的构造方法通过setState(-1)抑制了线程中断，这里通过unlock允许中断
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        //首先执行worker中的firstTask，执行完以后循环地从workerQueue中获取任务执行
        //第二次循环时task==null，此时会通过getTask方法从工作队列中取出任务
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            /**
            * 如果线程池的状态是stop、tidying、terminated，线程池不再执行任务，确保任务被中断
            * 如果线程池是running、shutdown，确保线程不被中断
            */
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                //钩子方法，由子类自定义实现
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        //执行到此处说明该工作线程自身既没有携带任务，也没有从工作队列中获取任务
        completedAbruptly = false;
    } finally {
        //处理工作线程的退出工作
        processWorkerExit(w, completedAbruptly);
    }
}
```

