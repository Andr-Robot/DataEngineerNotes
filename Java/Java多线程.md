[toc]

# 线程状态转换
![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81%E8%BD%AC%E6%8D%A2%E5%85%B3%E7%B3%BB.png)

1. **新建**(New)：创建后尚未启动的线程处于这种状态。
2. **运行**(Runable)：Runable包括了操作系统线程状态中的Running和Ready，也就是处于此状态的线程有可能正在执行，也有可能正在等待着CPU为它分配执行时间。
3. **无限期等待**(Waiting)：处于这种状态的线程不会被分配CPU执行时间，它们要等待被其他线程显式地唤醒。
    
    进入方法 | 退出方法
    ---|---
    没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll()
    没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕
    LockSupport.park() 方法 | -

4. **限期等待**(Timed Waiting)：处于这种状态的线程也不会被分配CPU执行时间，不过**无须等待被其他线程显式地唤醒，在一定时间之后它们会由系统自动唤醒**。

    进入方法 | 退出方法
    ---|---
    Thread.sleep() 方法 | 时间结束
    设置了 Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll()
    设置了 Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕
    LockSupport.parkNanos() 方法 | -
    LockSupport.parkUntil() 方法 | -

5. **阻塞**(Blocked)：该状态程序在等待获取一个排他锁，如果其线程释放了锁就会结束此状态。程序在同步时会在该状态。
6. **结束**(Terminated)：已终止线程的线程状态，线程已经结束执行。

# 使用多线程
有三种使用线程的方法：
- 实现 Runnable 接口；
- 实现 Callable 接口；
- 继承 Thread 类。

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以说任务是通过线程驱动从而执行的。

## 实现 Runnable 接口
需要实现 **run()** 方法。通过 Thread 调用 start() 方法来启动线程。

```java
public class MyRunnable implements Runnable {
    public void run() {
        // ...
    }
}
```

```java
public static void main(String[] args) {
    MyRunnable instance = new MyRunnable();
    Thread thread = new Thread(instance);
    thread.start();
}
```

## 实现 Callable 接口
Callable 位于`java.util.concurrent`包下，它是一个接口，只声明了一个叫做call()的方法。

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

Callable 接口类似于Runnable，两者都是为那些其实例可能被另一个线程执行的类设计的。和 Runnable 接口中的`run()`方法类似，Callable 提供一个`call()`方法作为线程的执行体。但是`call()`方法比`run()`方法更加强大，这要体现在：
1. call 方法可以**有返回值**。返回值的类型即是 Callable 接口传递进来的V类型。
2. call 方法**可以声明抛出异常**。

与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装。

```java
public class MyCallable implements Callable<Integer> {
    public Integer call() {
        return 123;
    }
}
```

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
    thread.start();
    System.out.println(ft.get());
}
```

## 继承 Thread 类
同样也是需要实现 run() 方法，因为 Thread 类也实现了 Runable 接口。

当调用 start() 方法启动一个线程时，虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的 run() 方法。

```java
public class MyThread extends Thread {
    public void run() {
        // ...
    }
}
```

```java
public static void main(String[] args) {
    MyThread mt = new MyThread();
    mt.start();
}
```

## 实现接口 VS 继承 Thread
实现接口会更好一些，因为：
- Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口；
- 类可能只要求可执行就行，继承整个 Thread 类开销过大。

## Callable和Future出现的原因
参见[Java程序员必须掌握的线程知识-Callable和Future](http://www.imooc.com/article/14383?block_id=tuijian_wz)     
Callable和Future，通过它们可以在任务执行完毕之后得到任务执行结果。

### Future接口
Future 接口位于`java.util.concurrent`包下，是Java 1.5中引入的接口。
Future主要用来对具体的Runnable或者Callable任务的执行结果进行取消、查询是否完成、获取结果。必要时可以通过`get()`方法获取执行结果，`get()`方法会阻塞直到任务返回结果。


```
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    
    boolean isCancelled();
    
    boolean isDone();
    
    V get() throws InterruptedException, ExecutionException;
    
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

因为**Future只是一个接口**，所以是无法直接用来创建对象使用的，因此就有了的FutureTask。

在Future接口中声明了5个方法，下面依次解释每个方法的作用：
- **cancel方法**：用来取消任务，如果取消任务成功则返回true，如果取消任务失败则返回false。参数mayInterruptIfRunning表示是否允许取消正在执行却没有执行完毕的任务，如果设置true，则表示可以取消正在执行过程中的任务。如果任务已经完成，则无论mayInterruptIfRunning为true还是false，此方法肯定返回false，即如果取消已经完成的任务会返回false；如果任务正在执行，若mayInterruptIfRunning设置为true，则返回true，若mayInterruptIfRunning设置为false，则返回false；如果任务还没有执行，则无论mayInterruptIfRunning为true还是false，肯定返回true。
- **isCancelled方法**：表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true。
- **isDone方法**：表示任务是否已经完成，若任务完成，则返回true；
- **get()方法**：用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；
- **get(long timeout, TimeUnit unit)**：用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。

也就是说Future提供了三种功能：
- 判断任务是否完成；
- 能够中断任务；
- 能够获取任务执行结果

# 基础线程机制
## Executor
Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

主要有三种 Executor：
- CachedThreadPool：一个任务创建一个线程；
- FixedThreadPool：所有任务只能使用固定大小的线程；
- SingleThreadExecutor：相当于大小为 1 的 FixedThreadPool。

```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < 5; i++) {
        executorService.execute(new MyRunnable());
    }
    executorService.shutdown();
}
```

## Daemon
在Java线程中，一种是用户线程，另一种是守护线程。

守护线程是程序运行时在后台提供服务的线程，不属于程序中不可或缺的部分。

当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。

典型的守护线程就是**垃圾回收线程**，当进程中没有非守护线程了，则垃圾回收线程也就没有存在必要了，自动销毁。

main() 属于非守护线程。

使用 setDaemon() 方法将一个线程设置为守护线程。

```java
public static void main(String[] args) {
    Thread thread = new Thread(new MyRunnable());
    thread.setDaemon(true);
}
```

## sleep()
Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

sleep() 可能会抛出 InterruptedException，因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地进行处理。

```java
public void run() {
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

## yield()
yield是把当前线程的执行权交出去，把当前线程由running变成runnable状态。但是放弃的时间不确定，有可能刚刚放弃，马上又获取CPU时间片。

对静态方法 Thread.yield() 的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。

```java
public void run() {
    Thread.yield();
}
```



# 停止线程
## 相关知识
在java中有以下3种方法可以终止正在运行的线程：
- 使用退出标志，使线程正常退出，也就是当run方法完成后线程终止。
- 使用stop方法强行终止，但是不推荐这个方法，因为stop和suspend及resume一样都是过期作废的方法。使用stop会带来数据不一致问题。
- 使用interrupt方法中断线程。

**interrupt()**：    
该方法只是在目标线程中设置一个标志，表示它已经被中断，并立即返回。这里需要注意的是，如果只是单纯的调用`interrupt()`方法，线程并没有实际被中断，会继续往下执行。

**interrupted() 和 isInterrupted()区别**：   
`interrupted()` 和 `isInterrupted()`都能够用于检测对象的“中断标记”。    
区别是，`interrupted()`除了返回中断标记之外，它还会清除中断标记(即将中断标记设为false)；而`isInterrupted()`仅仅返回中断标记。

```java
public static boolean interrupted() {
    return currentThread().isInterrupted(true);
}

public boolean isInterrupted() {
    return isInterrupted(false);
}
```

## InterruptedException
通过调用一个线程的 interrupt() 来中断该线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException，从而提前结束该线程。但是不能中断 I/O 阻塞和 synchronized 锁阻塞。

对于以下代码，在 main() 中启动一个线程之后再中断它，由于线程中调用了 Thread.sleep() 方法，因此会抛出一个 InterruptedException，从而提前结束线程，不执行之后的语句。

```java
public class InterruptExample {

    private static class MyThread1 extends Thread {
        @Override
        public void run() {
            try {
                Thread.sleep(2000);
                System.out.println("Thread run");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
public static void main(String[] args) throws InterruptedException {
    Thread thread1 = new MyThread1();
    thread1.start();
    thread1.interrupt();
    System.out.println("Main run");
}

输出：
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at InterruptExample.lambda$main$0(InterruptExample.java:5)
    at InterruptExample$$Lambda$1/713338599.run(Unknown Source)
    at java.lang.Thread.run(Thread.java:745)
```

## interrupted()
如果一个线程的 run() 方法执行一个无限循环，并且没有执行 sleep() 等会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法就无法使线程提前结束。

但是调用 interrupt() 方法会设置线程的中断标记，此时调用 interrupted() 方法会返回 true。因此可以在循环体中使用 interrupted() 方法来判断线程是否处于中断状态，从而提前结束线程。

```java
public class InterruptExample {

    private static class MyThread2 extends Thread {
        @Override
        public void run() {
            while (!interrupted()) {
                // ..
            }
            System.out.println("Thread end");
        }
    }
}
public static void main(String[] args) throws InterruptedException {
    Thread thread2 = new MyThread2();
    thread2.start();
    thread2.interrupt();
}

输出：
Thread end
```

## Executor 的中断操作
调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 shutdownNow() 方法，则相当于调用每个线程的 interrupt() 方法。

以下使用 Lambda 创建线程，相当于创建了一个匿名内部线程。

```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> {
        try {
            Thread.sleep(2000);
            System.out.println("Thread run");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
    executorService.shutdownNow();
    System.out.println("Main run");
}

输出：
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at ExecutorInterruptExample.lambda$main$0(ExecutorInterruptExample.java:9)
    at ExecutorInterruptExample$$Lambda$1/1160460865.run(Unknown Source)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at java.lang.Thread.run(Thread.java:745)
```

如果只想中断 Executor 中的一个线程，可以通过使用 submit() 方法来提交一个线程，它会返回一个 Future<?> 对象，通过调用该对象的 **cancel(true)** 方法就可以中断线程。

```java
Future<?> future = executorService.submit(() -> {
    // ..
});
future.cancel(true);
```

# 线程优先级
优先级越高的线程得到的CPU资源越多，也就是CPU优先执行优先级较高的线程对象中的任务。

设置优先级使用`setPriority()`方法。

Java线程优先级分为1-10这10个等价，如果小于1或者大于10，会抛出IllegalArgumentExceptio。

```java
public final static int MIN_PRIORITY = 1;

public final static int NORM_PRIORITY = 5;

public final static int MAX_PRIORITY = 10;

public final void setPriority(int newPriority) {
    ThreadGroup g;
    checkAccess();
    if (newPriority > MAX_PRIORITY || newPriority < MIN_PRIORITY) {
        throw new IllegalArgumentException();
    }
    if((g = getThreadGroup()) != null) {
        if (newPriority > g.getMaxPriority()) {
            newPriority = g.getMaxPriority();
        }
        setPriority0(priority = newPriority);
    }
}
```

线程具有继承性，A线程启动B线程，则B线程优先级和A线程一样。

- 优先级具有规则性：CPU尽量将执行资源让给优先级高的先执行。
- 优先级具有随机性：优先级高的线程不一定每次都先执行完run方法中的任务。

# 对象及变量的并发访问
## synchronized
- 方法中的变量不存在非线程安全问题，永远是线程安全的。这是方法内部变量是私有的特性造成的。
- 多个线程共同访问一个对象中的实例变量，会存在非线程安全问题，需要给操作实例变量的方法加关键字（synchronized），使之成为同步方法。
- synchronized取得的锁是**对象锁**，如果多个线程访问同一个对象，则只创建一个锁，哪个线程先得到锁，其他线程只能等待；如果多个线程访问多个对象，则JVM会创建多个锁。
- 调用synchronized声明的方法**一定是排队运行**的。
- **脏读是指在读取实例变量时，此值已经被其他线程更改过**，解决方法就是使用synchronized关键字。
- 当线程出现异常，其所持有的锁会自动释放。
- 多个线程同时持有锁对象，同步；分别持有锁对象，异步。

**关于synchronized的两条规定**：
- 线程解锁前，必须把共享变量的最新值刷新到主内存中
- 线程加锁时，将清空工作内存中共享变量的值，从而使用共享变量时需要从主内存中重新读取最新的值（注意：加锁和解锁需要是同一把锁）

**注：线程解锁前对共享变量的修改在下次加锁时对其他线程可见**

### 线程执行互斥代码的过程
1. 获得互斥锁
2. 清空工作内存
3. 从主内存拷贝变量的最新副本到工作内存
4. 执行代码
5. 将更改后的共享变量的值刷新到主内存
6. 释放互斥锁

### 同步一个代码块

```java
public void func() {
    synchronized (this) {
        // ...
    }
}
```

**它只作用于同一个对象**，如果调用两个对象上的同步代码块，就不会进行同步。**同步代码块不能使用String作为锁对象**。

对于以下代码，使用 ExecutorService 执行了两个线程，由于调用的是同一个对象的同步代码块，因此这两个线程会进行同步，当一个线程进入同步语句块时，另一个线程就必须等待。

```java
public class SynchronizedExample {

    public void func1() {
        synchronized (this) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}

public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func1());
    executorService.execute(() -> e1.func1());
}

输出：
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

对于以下代码，两个线程调用了不同对象的同步代码块，因此这两个线程就不需要同步。从输出结果可以看出，两个线程交叉执行。

```java
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    SynchronizedExample e2 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func1());
    executorService.execute(() -> e2.func1());
}

输出：
0 0 1 1 2 2 3 3 4 4 5 5 6 6 7 7 8 8 9 9
```

**锁非this对象**时，如果一个类中有synchronized方法和synchronized(非this)代码块，则在执行的时候是异步的，不与其他锁this同步方法争抢this锁。但同一时间依然只有一个线程可以执行synchronized(非this)代码块。

### 同步一个方法

```java
public synchronized void func () {
    // ...
}
```

它和同步代码块一样，**作用于同一个对象**。

### 同步一个类

```java
public void func() {
    synchronized (SynchronizedExample.class) {
        // ...
    }
}
```

**作用于整个类**，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。

```java
public class SynchronizedExample {

    public void func2() {
        synchronized (SynchronizedExample.class) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}

public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    SynchronizedExample e2 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func2());
    executorService.execute(() -> e2.func2());
}

输出：
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

### 同步一个静态方法

```java
public synchronized static void fun() {
    // ...
}
```

**作用于整个类**。

### synchronized锁重入
synchronized拥有锁重入机制，也就是**在使用synchronized时，当一个线程得到对象锁后，再次请求锁时可以再次得到该对象的锁**。这也证明了在synchronized方法或方法块的内部调用synchronized方法或方法块时，是永远可以得到锁的。

当存在父子类继承关系时，**子类完全可以通过“可重入锁”调用父类的同步方法**。

```java
public class FClass {
	public int i = 10;
	synchronized public void operationF() {
		i --;
		System.out.println("Fclass print i=" + i);
		try {
			Thread.sleep(100);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}

public class SubClass extends FClass {
	synchronized public void operationSub() {
		while (i > 0) {
			i --;
			System.out.println("SubClass print i=" + i);
			try {
				Thread.sleep(100);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			this.operationF();
		}
	}
}

public class MyThread extends Thread {
	@Override
	public void run() {
		SubClass sub = new SubClass();
		sub.operationSub();
	}
}

public class Test {

	public static void main(String[] args) {
		MyThread mThread = new MyThread();
		mThread.start();
	}
}
```

### 监测是否出现死锁
**CMD --> jps --> jstack -l 进程Id值**


## ReentrantLock
ReentrantLock 是 java.util.concurrent（J.U.C）包中的锁。

```java
public class LockExample {

    private Lock lock = new ReentrantLock();

    public void func() {
        lock.lock();
        try {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        } finally {
            lock.unlock(); // 确保释放锁，从而避免发生死锁。
        }
    }
}

public static void main(String[] args) {
    LockExample lockExample = new LockExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> lockExample.func());
    executorService.execute(() -> lockExample.func());
}

输出：
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```


## volatile
**主要作用是使变量在多个线程间可见。** 关键字volatile强制从公共堆栈中取得变量的值，而不是从线程私有数据栈中取得变量的值。

![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/volatile.png)

锁提供了两种主要特性：**互斥（mutual exclusion） 和可见性（visibility）**。
- **互斥**即一次只允许一个线程持有某个特定的锁，因此可使用该特性实现对共享数据的协调访问协议，这样，一次就只有一个线程能够使用该共享数据。
- **可见性**要更加复杂一些，它必须确保释放锁之前对共享数据做出的更改对于随后获得该锁的另一个线程是可见的 —— 如果没有同步机制提供的这种可见性保证，线程看到的共享变量可能是修改前的值或不一致的值，这将引发许多严重问题。

**Volatile 变量具有 synchronized 的可见性特性，但是不具备原子特性**，可以通过加锁（`synchronized`或`java.util.concurrent`中的原子类）来实现原子性。这就是说线程能够自动发现 volatile 变量的最新值。Volatile 变量可用于提供线程安全。

**volatile 变量不会像锁那样造成线程阻塞**，在某些情况下，**如果读操作远远大于写操作，volatile 变量还可以提供优于锁的性能优势**。

**使用条件**：您只能在有限的一些情形下使用 volatile 变量替代锁。要使 volatile 变量提供理想的线程安全，必须同时满足下面两个条件：
- 对变量的写操作不依赖于当前值。 
- 该变量没有包含在具有其他变量的不变式中。

## volatile vs synchronized
1. volatile是线程同步的轻量级实现，所以volatile性能肯定比synchronized要好。
2. volatile只能修饰变量，synchronized可以修饰方法和代码块。
3. 多线程访问volatile不会阻塞，而synchronized会出现阻塞。
4. volatile保证数据可见性，但不保证原子性。synchronized保证原子性，可以间接保证可见性，它可以将私有内存和公共内存的数据做同步。
5. volatile解决的是变量在多个线程之间的可见性，而synchronized解决的是多个线程之间访问资源的同步性。

**线程安全包括原子性和可见性两个方面。** 

## synchronized vs ReentrantLock
### 锁的实现
synchronized 是 JVM 实现的，而 ReentrantLock 是 JDK 实现的。   
原始的synchronized的实现是悲观锁，Lock的实现是乐观锁。

### 性能
新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。

### 释放锁
**synchronized在发生异常时，会自动释放线程占有的锁**，因此不会导致死锁现象发生；而Lock在发生异常时，如果**没有主动通过unLock()去释放锁**，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁。

### 等待可中断
当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

ReentrantLock 可中断，而 synchronized 不行。

### 是否获得成功锁
通过Lock可以知道有没有成功获取锁，而synchronized却无法办到。

### 公平锁
公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

### 锁绑定多个条件
一个 ReentrantLock 可以同时绑定多个 Condition 对象。

### 多线程读操作
Lock可以提高多个线程进行读操作的效率。而 synchronized 不行。

### 使用选择
除非需要使用 ReentrantLock 的高级功能，否则优先使用 synchronized。这是因为 synchronized 是 JVM 实现的一种锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 JVM 会确保锁的释放。

# CAS
CAS（Compare and swap）比较和替换是设计并发算法时用到的一种技术。简单来说，比较和替换是使用一个期望值和一个变量的当前值进行比较，如果当前变量的值与我们期望的值相等，就使用一个新值替换当前变量的值。

Java5以来，你可以使用java.util.concurrent.atomic包中的一些原子类来使用CPU中的这些功能：

```java
private AtomicBoolean locked = new AtomicBoolean(false);
public boolean lock() {
    return locked.compareAndSet(false, true);
}

public final boolean compareAndSet(boolean expect, boolean update) {
    int e = expect ? 1 : 0;
    int u = update ? 1 : 0;
    return unsafe.compareAndSwapInt(this, valueOffset, e, u);
}
```

locked变量不再是boolean类型而是AtomicBoolean。这个类中有一个compareAndSet()方法，它使用一个期望值和AtomicBoolean实例的值比较，和两者相等，则使用一个新值替换原来的值。在这个例子中，它比较locked的值和false，如果locked的值为false，则把修改为true。

如果值被替换了，compareAndSet()返回true，否则，返回false。

CAS的ABA问题：
- 进程P1在共享变量中读到值为A
- P1被抢占了，进程P2执行
- P2把共享变量里的值从A改成了B，再改回到A，此时被P1抢占。
- P1回来看到共享变量里的值没有被改变，于是继续执行。

## 和lock的区别
使用lock的方式(**synchronized关键字或Lock类**等)是一种**悲观锁机制**。但是大多数情况下并没有大量的竞争，相对而言**CAS是一种乐观策略**，认为竞争很少出现，当竞争发生时抛给调用方处理重试还是其他处理方式，由于没有加锁带来的较高开销和加锁中的临界区限制，这种**无锁(lock-free)机制**相比加锁具有更高的扩展性。

## CAS常见使用场景
**常见的使用场景是替换锁进行数据修改**，常见的使用方法是获取当前值，根据当前值计算出要设置的值，然后使用CAS尝试交换，如果成功返回，如果失败可以选择重试或者采取其他策略。




# 参考文献
[java多线程之内存可见性-synchronized、volatile](https://www.cnblogs.com/zhilu-doc/p/5778180.html)    
[Java中CAS的使用和实现分析](https://liuzhengyang.github.io/2017/05/11/cas/)    
[深入浅出CAS](https://www.jianshu.com/p/fb6e91b013cc)    


