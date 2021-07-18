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

### 增强的Executor----ExecutorService  

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

### 周期任务的调度----ScheduledExecutorService  

在ExecutorService的基础上，J.U.C又提供了一个接口ScheduledExecutorService，该接口主要是为了满足某些任务能够定时执行或者周期性的执行  

```java
public interface ScheduledExecutorService extends ExecutorService
```

ScheduledExecutorService提供了一系列schedule方法，可以在给定的延迟后执行提交的任务，或者每隔指定的周期执行一次任务  

```java
public interface ScheduledExecutorService extends ExecutorService {
 
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
}
```

以上就是Executor框架中三个最核心的接口，这三个接口的关系如下图  

![clipboard.png](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/bVbh0SH)  

## 生产Executor的工厂  

通过上一部分，我们知道Executors框架就是同来解耦任务本身与任务的执行的，并提供了三个核心的方法来满足开发人员的需求：  

+ Executor：提交普通的可执行任务

+ ExecutorService：提供对线程池生命周期的管理、异步任务的支持

+ ScheduledExecutorService：提供对任务的周期性支持

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

![Executors框架](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/Executors%E6%A1%86%E6%9E%B6.png)  

