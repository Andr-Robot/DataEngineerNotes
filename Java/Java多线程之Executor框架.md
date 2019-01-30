* [线程池](#线程池)
* [线程池的生命周期](#线程池的生命周期)
    * [启动线程池](#启动线程池)
    * [关闭线程池](#关闭线程池)
    * [线程池结束](#线程池结束)
    * [总结](#总结)
* [为什么要引入Executor框架](#为什么要引入executor框架)
* [Executor接口](#executor接口)
* [ExecutorService接口](#executorservice接口)
* [ThreadPoolExecutor](#threadpoolexecutor)
    * [ThreadPoolExecutor构造函数](#threadpoolexecutor构造函数)
* [Executors](#executors)
    * [newCachedThreadPool](#newcachedthreadpool)
    * [newFixedThreadPool](#newfixedthreadpool)
    * [newSingleThreadExecutor](#newsinglethreadexecutor)
    * [newScheduledThreadPool](#newscheduledthreadpool)
    * [newSingleThreadScheduledExecutor](#newsinglethreadscheduledexecutor)
    * [总结](#总结)
        * [newSingleThreadExecutor 与 newFixedThreadPool(1) 的区别](#newsinglethreadexecutor-与-newfixedthreadpool1-的区别)
* [ScheduledExecutorService](#scheduledexecutorservice)
* [总结](#总结)
    * [Executor vs ExecutorService vs Executors](#executor-vs-executorservice-vs-executors)
    * [饱和策略](#饱和策略)
    * [Runable接口和Callable接口](#runable接口和callable接口)
* [参考文献](#参考文献)

# 线程池
线程池就是**限制系统中使用线程的数量以及更好的使用线程**。根据系统的运行情况，可以自动或手动设置线程数量，达到运行的最佳效果：配置少了，将影响系统的执行效率，配置多了，又会浪费系统的资源。

当一个任务执行完毕后，就从队列中取一个新任务运行，如果没有新任务，那么这个线程将等待。如果来了一个新任务，但是没有空闲线程的话，那么把任务加入到等待队列中。

# 线程池的生命周期
参见[深入浅出 Java Concurrency (30): 线程池 part 3 Executor 生命周期](http://www.blogjava.net/xylz/archive/2011/01/04/342316.html)     

线程是有多种执行状态的，同样管理线程的线程池也有多种状态。JVM会在所有线程（非后台daemon线程）全部终止后才退出，为了节省资源和有效释放资源关闭一个线程池就显得很重要。有时候无法正确的关闭线程池，将会阻止JVM的结束。

**线程池Executor是异步的执行任务**，因此任何时刻不能够直接获取提交的任务的状态。这些任务有可能已经完成，也有可能正在执行或者还在排队等待执行。因此关闭线程池可能出现一下几种情况：
- 平缓关闭：已经启动的任务全部执行完毕，同时不再接受新的任务
- 立即关闭：取消所有正在执行和未执行的任务

## 启动线程池
线程池在构造前（new操作）是初始状态，一旦构造完成**线程池就进入了执行状态RUNNING**。严格意义上讲线程池构造完成后并没有线程被立即启动，只有进行“预启动”或者接收到任务的时候才会启动线程。

线程池是出于运行状态，随时准备接受任务来执行。

## 关闭线程池
线程池运行中可以通过shutdown()和shutdownNow()来改变运行状态。
- shutdown()是一个平缓的关闭过程，线程池停止接受新的任务，同时等待已经提交的任务执行完毕，包括那些进入队列还没有开始的任务，这时候**线程池处于SHUTDOWN状态**；
- shutdownNow()是一个立即关闭过程，线程池停止接受新的任务，同时线程池取消所有执行的任务和已经进入队列但是还没有执行的任务，这时候**线程池处于STOP状态**。**shutdownNow方法本质是调用Thread.interrupt()方法。但我们知道该方法仅仅是让线程处于interrupted状态，并不会让线程真正的停止！所以若只调用或只调用一次shutdownNow()方法，不一定会让线程池中的线程都关闭掉，线程中必须要有处理interrupt事件的机制。**

## 线程池结束
一旦shutdown()或者shutdownNow()执行完毕，**线程池就进入TERMINATED状态**，此时线程池就结束了。
- isTerminating() 如果关闭后所有任务都已完成，则返回 true。
- isShutdown() 如果此执行程序已关闭，则返回 true。

![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/Executor-Lifecycle_thumb_1.png)

## 总结
1. 线程池有运行、关闭、停止、结束四种状态，结束后就会释放所有资源
2. 平缓关闭线程池使用shutdown()
3. 立即关闭线程池使用shutdownNow()，同时得到未执行的任务列表
4. 检测线程池是否正处于关闭中，使用isShutdown()
5. 检测线程池是否已经关闭使用isTerminated()
6. 定时或者永久等待线程池关闭结束使用awaitTermination()操作

# 为什么要引入Executor框架
如果使用`new Thread(...).start()`的方法处理多线程，有如下缺点：
- **开销大**。对于JVM来说，每次新建线程和销毁线程都会有很大的开销。
- **线程缺乏管理**。没有一个池来限制线程的数量，如果并发量很高，会创建很多的线程，而且线程之间可能会有相互竞争，这将会过多得占用系统资源，增加系统资源的消耗量。而且线程数量超过系统负荷，容易导致系统不稳定。

使用线程池的方式，有如下优点：
- **复用线程**。通过复用创建的了的线程，减少了线程的创建、消亡的开销。
- **有效控制并发线程数**。
- **提供了更简单灵活的线程管理**。可以提供定时执行、定期执行、单线程、可变线程数等多种线程使用功能。

# Executor接口
Executor是一个接口，**它将任务的提交与任务的执行分离开来，定义了一个接收Runnable对象的方法`execute`。**

```java
public interface Executor {
    void execute(Runnable command);
}
```

# ExecutorService接口
ExecutorService继承了Executor，是一个比Executor使用更广泛的子类接口。定义了**终止任务、提交任务、跟踪任务返回结果**等方法。

一个ExecutorService是可以关闭的，关闭之后它将不能再接收任何任务。对于不再使用的ExecutorService，应该将其关闭以释放资源。

```java
public interface ExecutorService extends Executor {
    /**
     * 平滑地关闭线程池，已经提交到线程池中的任务会继续执行完。
     */
    void shutdown();
    /**
     * 立即关闭线程池，返回还没有开始执行的任务列表。
     * 会尝试中断正在执行的任务（每个线程调用 interruput方法），但这个行为不一定会成功。
     */
    List<Runnable> shutdownNow();
    /**
     * 判断线程池是否已经关闭
     */
    boolean isShutdown();
    /**
     * 判断线程池的任务是否已经执行完毕。
     * 注意此方法调用之前需要先调用shutdown()方法或者shutdownNow()方法，否则总是会返回false
     */
    boolean isTerminated();
    /**
     * 判断线程池的任务是否都执行完。
     * 如果没有任务没有执行完毕则阻塞，直至任务完成或者达到了指定的timeout时间就会返回
     */
    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;
    /**
     * 提交带有一个返回值的任务到线程池中去执行（回调），返回的 Future 表示任务的待定结果。
     * 当任务成功完成后，通过 Future 实例的 get() 方法可以获取该任务的结果。
     * Future 的 get() 方法是会阻塞的。
     */
    <T> Future<T> submit(Callable<T> task);
    /**
     *提交一个Runnable的任务，当任务完成后，可以通过Future.get()获取的是提交时传递的参数T result
     */
    <T> Future<T> submit(Runnable task, T result);
    /**
     * 提交一个Runnable的任务，它的Future.get()得不到任何内容，它返回值总是Null。
     * 为什么有这个方法？为什么不直接设计成void submit(Runnable task)这种方式？
     * 这是因为Future除了get这种获取任务信息外，还可以控制任务，
     具体体现在 Future的这个方法上：boolean cancel(boolean mayInterruptIfRunning)
     这个方法能够去取消提交的Rannable任务。
     */
    Future<?> submit(Runnable task);
    /**
     * 执行一组给定的Callable任务，返回对应的Future列表。列表中每一个Future都将持有该任务的结果和状态。
     * 当所有任务执行完毕后，方法返回，此时并且每一个Future的isDone()方法都是true。
     * 完成的任务可能是正常结束，也可以是异常结束
     * 如果当任务执行过程中，tasks集合被修改了，那么方法的返回结果将是不确定的，
       即不能确定执行的是修改前的任务，还是修改后的任务
     */
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;
    /**
     * 执行一组给定的Callable任务，返回对应的Future列表。列表中每一个Future都将持有该任务的结果和状态。
     * 当所有任务执行完毕后或者超时后，方法将返回，此时并且每一个Future的isDone()方法都是true。
     * 一旦方法返回，未执行完成的任务被取消，而完成的任务可能正常结束或者异常结束， 
     * 完成的任务可以是正常结束，也可以是异常结束
     * 如果当任务执行过程中，tasks集合被修改了，那么方法的返回结果将是不确定的
     */
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;
    /**
     * 执行一组给定的Callable任务，当成功执行完（没抛异常）一个任务后此方法便返回，返回的是该任务的结果
     * 一旦此正常返回或者异常结束，未执行的任务都会被取消。 
     * 如果当任务执行过程中，tasks集合被修改了，那么方法的返回结果将是不确定的
     */
    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;
    /**
     * 执行一组给定的Callable任务，当在timeout（超时）之前成功执行完（没抛异常）一个任务后此方法便返回，返回的是该任务的结果
     * 一旦此正常返回或者异常结束，未执行的任务都会被取消。 
     * 如果当任务执行过程中，tasks集合被修改了，那么方法的返回结果将是不确定的
     */
    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

# ThreadPoolExecutor
ThreadPoolExecutor是Executor框架最重要的一个类，它即是真正意义上的线程池。
## ThreadPoolExecutor构造函数

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
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

构造器的各个参数说明：
- **corePoolSize**：核心线程数，核心线程会一直存活，即使没有任务需要处理。但如果设置了allowCoreThreadTimeOut `为 true 则核心线程也会超时退出。
- **maximumPoolSize**：最大线程数，线程池中可允许创建的最大线程数。
- **keepAliveTime**：当线程池中的线程数大于核心线程数，那些多余的线程空闲时间达到keepAliveTime后就会退出，直到线程数量等于corePoolSize。如果设置了allowCoreThreadTimeout设置为true，则所有线程均会退出直到线程数量为0。
- **unit**：keepAliveTime参数的时间单位
- **workQueue**：在任务执行前用来保存任务的 阻塞队列。这个队列只会保存通过execute方法提交到线程池的Runnable任务。在ThreadPolExecutor线程池的API文档中，一共推荐了三种等待队列，它们是：SynchronousQueue、LinkedBlockingQueue 和 ArrayBlockingQueue。
- **threadFactory**：线程池创建新线程时使用的factory。默认使用defaultThreadFactory创建线程。
- **handle**：饱和策略。当线程池的线程数已达到最大，并且任务队列已满时来处理被拒绝任务的策略。默认使用ThreadPoolExecutor.AbortPolicy，任务被拒绝时将抛出RejectExecutorException

除此之外，ThreadPoolExecutor还有两个个常用的参数设置：
- allowCoreThreadTimeout：是否允许核心线程空闲退出，默认值为false。
- queueCapacity：任务队列的容量。

ThreadPoolExecutor线程池的逻辑结构图:

![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/2489678458-5b016b3ae7714_articlex.png)

线程池执行任务的行为方式：
1. 当线程数小于核心线程数时，创建线程。
2. 当线程数大于等于核心线程数，且任务队列未满时，将任务放入任务队列。
3. 当线程数大于等于核心线程数，且任务队列已满
    1. 若线程数小于最大线程数，创建线程
    2. 若线程数等于最大线程数，抛出异常，拒绝任务

# Executors
Executors类是一个工厂类，提供工厂方法来创建不同类型的线程池，比如`FixedThreadPool` 或 `CachedThreadPool`。

## newCachedThreadPool
创建一个**不限制线程数量**的动态线程池。
- 因为有多个线程存在，任务不一定会按照顺序执行。
- 一个线程完成任务后，空闲时间达到60秒则会被结束。
- 在执行新的任务时，当线程池中有之前创建的空闲线程就使用这个线程，否则就新建一条线程。

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

可以看到`newCachedThreadPool`使用的队列是`SynchronousQueue`。线程池的线程数可达到`Integer.MAX_VALUE`，即2147483647。此外由于会有线程的创建和销毁，所以会有一定的系统开销。

## newFixedThreadPool
创建一个**可重用的固定线程数量**的线程池。即`corePoolSize=线程池中的线程数= maximumPoolSize`。
- 如果没有任务执行，所有的线程都将等待。
- 如果线程池中的所有线程都处于活动状态，此时再提交任务就在队列中等待，直到有可用线程。
- 如果线程池中的某个线程由于异常而结束时，线程池就会再补充一条新线程。

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

## newSingleThreadExecutor
创建一个**单线程的线程池**：启动一个线程负责按顺序执行任务，先提交的任务先执行。也就是相当于单线程串行执行所有任务。

其原理是：任务会被提交到一个队列里，启动的那个线程会从队里里取任务，然后执行，执行完，再从队列里取下一个任务，再执行。如果该线程执行一个任务失败，并导致线程结束，系统会创建一个新的线程去执行队列里后续的任务，不会因为前面的任务有异常导致后面无辜的任务无法执行。

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

## newScheduledThreadPool
创建一个固定线程数的`ScheduledExecutorService`对象，在指定延时之后执行或者以固定的频率周期性的执行提交的任务。

```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
```

## newSingleThreadScheduledExecutor
创建一个单线程的`ScheduledExecutorService`，在指定延时之后执行或者以固定的频率周期性的执行提交的任务。在线程池关闭之前如果有一个任务执行失败，并导致线程结束，系统会创建一个新的线程接着执行队列里的任务。

```java
public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
    return new DelegatedScheduledExecutorService
        (new ScheduledThreadPoolExecutor(1));
}
```

## 总结
### newSingleThreadExecutor 与 newFixedThreadPool(1) 的区别

```java
((ThreadPoolExecutor)newFixedThreadPool(1)).setCorePoolSize(3);
```

`newFixedThreadPool(1)`可以后期修改线程数，不保证线程只有一个。而`newSingleThreadExecutor`可以保证。

# ScheduledExecutorService
ScheduledExecutorService是一个线程池，用来在指定延时之后执行或者以固定的频率周期性的执行提交的任务。它包含了4个方法。

```java
public interface ScheduledExecutorService extends ExecutorService {

    /**
     * 在指定delay（延时）之后，执行提交Runnable的任务，返回一个ScheduledFuture，
     * 任务执行完成后ScheduledFuture的get()方法返回为null，ScheduledFuture的作用是可以cancel任务
     */
    public ScheduledFuture<?> schedule(Runnable command,
                                       long delay, TimeUnit unit);

    /**
     * 在指定delay（延时）之后，执行提交Callable的任务，返回一个ScheduledFuture
     */
    public <V> ScheduledFuture<V> schedule(Callable<V> callable,
                                           long delay, TimeUnit unit);

    /**
     * 提交一个Runnable任务延迟了initialDelay时间后，开始周期性的执行该任务，每period时间执行一次
     * 如果任务异常则退出。如果取消任务或者关闭线程池，任务也会退出。
     * 如果任务执行一次的时间大于周期时间，则任务执行将会延后执行，而不会并发执行
     */
    public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                                  long initialDelay,
                                                  long period,
                                                  TimeUnit unit);

    /**
     * 提交一个Runnable任务延迟了initialDelay时间后，开始周期性的执行该任务，以后
       每两次任务执行的间隔是delay
     * 如果任务异常则退出。如果取消任务或者关闭线程池，任务也会退出。
     */
    public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                     long initialDelay,
                                                     long delay,
                                                     TimeUnit unit);

}
```

# 总结
## Executor vs ExecutorService vs Executors
这三者均是 Executor 框架中的一部分。
1. Executor 和 ExecutorService 这两个接口主要的区别是：**ExecutorService 接口继承了 Executor 接口，是 Executor 的子接口**
2. Executor 和 ExecutorService 第二个区别是：Executor 接口定义了 `execute()`方法用来接收一个Runnable接口的对象，而 ExecutorService 接口中的 `submit()`方法可以接受Runnable和Callable接口的对象。
3. Executor 和 ExecutorService 接口第三个区别是 Executor 中的 `execute()` 方法不返回任何结果，而 ExecutorService 中的 `submit()`方法可以通过一个 Future 对象返回运算结果。
4. Executor 和 ExecutorService 接口第四个区别是除了允许客户端提交一个任务，**ExecutorService 还提供用来控制线程池的方法**。比如：调用 shutDown() 方法终止线程池。
5. **Executors 类提供工厂方法用来创建不同类型的线程池**。比如: newSingleThreadExecutor() 创建一个只有一个线程的线程池，newFixedThreadPool(int numOfThreads)来创建固定线程数的线程池，newCachedThreadPool()可以根据需要创建新的线程，但如果已有线程是空闲的会重用已有线程。

## 饱和策略
当线程池中的线程均处于工作状态，并且线程数已达线程池允许的最大线程数时，就会采取指定的饱和策略来处理新提交的任务。总共有四种策略：
- AbortPolicy: 直接抛异常
- CallerRunsPolicy: 用调用者的线程来运行任务
- DiscardOldestPolicy: 丢弃线程队列里最近的一个任务，执行新提交的任务
- DiscardPolicy 直接将新任务丢弃

如果使用 Executors 的工厂方法创建的线程池，那么饱和策略都是采用**默认的 AbortPolicy**，所以如果我们想当线程池已满的情况，使用调用者的线程来运行任务，就要自己创建线程池，指定想要的饱和策略，而不是使用 Executors 了。

## Runable接口和Callable接口
参见[java多线程系列：Executors框架](https://juejin.im/post/5b1f21abf265da6e6414a8e3#heading-13)

那么就从提交任务入口看看吧

submit方法是由抽象类AbstractExecutorService实现的

```
public Future<?> submit(Runnable task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<Void> ftask = newTaskFor(task, null);
    execute(ftask);
    return ftask;
}

public <T> Future<T> submit(Callable<T> task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task);
    execute(ftask);
    return ftask;
}
```

可以看出将传入的Runnable对象和Callable传入一个newTaskFor方法，然后返回一个RunnableFuture对象

我们再来看看newTaskFor方法

```
protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
    return new FutureTask<T>(runnable, value);
}

protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
    return new FutureTask<T>(callable);
}
```

这里都是调用FutureTask的构造函数，我们接着往下看

```
private Callable<V> callable;

public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;      
}

public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       
}
```

FutureTask类中有个成员变量callable，而传入的Runnable对象则继续调用Executors工厂类的callable方法返回一个Callable对象

```
public static <T> Callable<T> callable(Runnable task, T result) {
    if (task == null)
        throw new NullPointerException();
    return new RunnableAdapter<T>(task, result);
}
//适配器
static final class RunnableAdapter<T> implements Callable<T> {
    final Runnable task;
    final T result;
    RunnableAdapter(Runnable task, T result) {
        this.task = task;
        this.result = result;
    }
    public T call() {
        task.run();
        return result;
    }
}
```

好了，到这里也就真相大白了，Runnable对象经过一系列的方法调用，最终被RunnableAdapter适配器适配成Callable对象。方法调用图如下

![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/submitexecutor.png)


# 参考文献
[Executor框架（一）Callable、Future、Executor和ExecutorService](https://segmentfault.com/a/1190000014940144)    
[Executor框架（二）Executors、ThreadPoolExecutor以及线程池执行任务的行为方式](https://segmentfault.com/a/1190000014944049)    
[Executor框架（三）ScheduledExecutorService-和-BlockingQueue](https://segmentfault.com/a/1190000015190796)     
[Java并发——线程池Executor框架](https://www.cnblogs.com/shijiaqi1066/p/3412300.html)    
[【译】Executor, ExecutorService 和 Executors 间的不同](https://yemengying.com/2017/03/17/difference-between-executor-executorService/)      
[Java多线程框架Executor详解](https://www.imooc.com/article/14377)     
[深入浅出 Java Concurrency (30): 线程池 part 3 Executor 生命周期](http://www.blogjava.net/xylz/archive/2011/01/04/342316.html)     
[java多线程系列：Executors框架](https://juejin.im/post/5b1f21abf265da6e6414a8e3#heading-13)   

