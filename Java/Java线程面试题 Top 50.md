* [wait()和sleep()的区别](#wait和sleep的区别)
* [notify()和notifyAll()的区别](#notify和notifyall的区别)
* [如何正确地停止一个线程](#如何正确地停止一个线程)
	* [interrupt()](#interrupt)
	* [interrupted() 和 isInterrupted()区别](#interrupted-和-isinterrupted区别)
* [start()和run()的区别](#start和run的区别)
* [this、Thread.currentThread()和this.currentThread()的区别](#thisthreadcurrentthread和thiscurrentthread的区别)

参见[Java线程面试题 Top 50](http://www.importnew.com/12773.html)    

# wait()和sleep()的区别
参见[java sleep和wait的区别的疑惑?](https://www.zhihu.com/question/23328075)和[Java中wait()与sleep()的区别](https://segmentfault.com/a/1190000002638478)    
![sleep vs wait](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/sleepwait.jpg)     
![线程状态图](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81%E8%BF%90%E8%A1%8C%E5%9B%BE.jpg)    
- sleep 让线程从 【running】 -> 【阻塞态】 时间结束/interrupt -> 【runnable】
- wait 让线程从 【running】 -> 【等待队列】notify -> 【锁池】 -> 【runnable】   

> 下列有关线程的说法正确的是：（ ）     
> a. 启动一个线程是调用start（）方法，是线程所代表的虚拟处理机处于可运行状态，这意味着线程此时就会立即运行。    
> b. notify（）方法可以确切的唤醒某个处于等待状态的线程。     
> c. wait（）方法可以使一个线程处于等待状态，但不会释放所持有对象的锁。     
> d. sleep（）方法使一个正在运行的线程处于睡眠状态，是一个静态方法，调用此方法时，需要捕捉InterruptedException异常    

解析：   
1. 调用run()才会运行
2. notify()随机唤醒、notifyAll()全部唤醒
3. wait()释放锁，进入等待序列
 
# notify()和notifyAll()的区别
参见[java中的notify和notifyAll有什么区别？](https://www.zhihu.com/question/37601861)和[java中notify和notifyall的区别](https://leokongwq.github.io/2017/02/22/java-notify-notifyall.html)   

- **锁池**：假设线程A已经拥有了某个对象(注意:不是类)的锁，而其它的线程想要调用这个对象的某个synchronized方法(或者synchronized块)，由于这些线程在进入对象的synchronized方法之前必须先获得该对象的锁的拥有权，但是该对象的锁目前正被线程A拥有，所以这些线程就进入了该对象的锁池中。
- **等待池**：假设一个线程A调用了某个对象的wait()方法，线程A就会释放该对象的锁后，进入到了该对象的等待池中。

区别：
- 如果线程调用了对象的 wait()方法，那么线程便会处于该对象的等待池中，等待池中的线程不会去竞争该对象的锁。
- 当有线程调用了对象的 **notifyAll()方法（唤醒所有等待该对象监视器的线程）或 notify()方法（只随机唤醒一个等待在该对象监视器上的线程）**，被唤醒的的线程便会进入该对象的锁池中，锁池中的线程会去竞争该对象锁。也就是说，调用了notify后只要一个线程会由等待池进入锁池，而notifyAll会将该对象等待池内的所有线程移动到锁池中，等待锁竞争。
- **被唤醒的线程不能马上运行**，直到当前线程释放了该对象的锁。被唤醒的线程以通常的方式来和其它正在想要获取该对象上锁的线程来竞争锁，并和其它线程处于同等地位，并没有什么特殊的地方。
- 优先级高的线程竞争到对象锁的概率大，假若某线程没有竞争到该对象锁，它还会留在锁池中，唯有线程再次调用 wait()方法，它才会重新回到等待池中。而竞争到对象锁的线程则继续往下执行，直到执行完了 synchronized 代码块，它会释放掉该对象锁，这时锁池中的线程会继续竞争该对象锁。



# 如何正确地停止一个线程
停止一个线程意味着在任务处理完任务之前停掉正在做的操作，也就是放弃当前的操作。停止一个线程可以用`Thread.stop()`方法，但最好不要用它。虽然它确实可以停止一个正在运行的线程，但是这个方法是不安全的，而且是已被废弃的方法。
在java中有以下3种方法可以终止正在运行的线程：
- 使用退出标志，使线程正常退出，也就是当run方法完成后线程终止。
- 使用stop方法强行终止，但是不推荐这个方法，因为stop和suspend及resume一样都是过期作废的方法。使用stop会带来数据不一致问题。
- 使用interrupt方法中断线程。

## interrupt()
该方法只是在目标线程中设置一个标志，表示它已经被中断，并立即返回。这里需要注意的是，如果只是单纯的调用`interrupt()`方法，线程并没有实际被中断，会继续往下执行。

## interrupted() 和 isInterrupted()区别
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

# start()和run()的区别
- start()方法被**用来启动新创建的线程，而且start()内部调用了run()方法**，这时无需等待run方法体代码执行完毕，可以直接继续执行下面的代码。通过调用Thread类的start()方法来启动一个线程，这时此线程是处于就绪状态，并没有运行。然后通过此Thread类调用方法run()来完成其运行操作的，这里方法run()称为线程体，它包含了要执行的这个线程的内容，Run方法运行结束，此线程终止。然后CPU再调度其它线程。start()只能调用一次，第二次调用start（）将在Java中抛出`IllegalThreadStateException`。
    注意：start()方法的顺序不代表线程启动的顺序。
- run()方法被调用的时候，**只会是在原来的线程中调用**，没有新的线程启动，程序还是要顺序执行，要等待run方法体执行完毕后，才可继续执行下面的代码。run()可以调用多次。

**调用start()**：

```java
public class MyThread extends Thread {
	@Override
	public void run() {
		// TODO Auto-generated method stub
		super.run();
		System.out.println("MyThread");
		System.out.println(Thread.currentThread().getName());
	}
}

public class Test {

	public static void main(String[] args) {
		MyThread mThread = new MyThread();
		System.out.println("--" + Thread.currentThread().getName());
		mThread.start();
		System.out.println("end!!!");
	}
}

输出：
--main
end!!!
MyThread
Thread-0
```

**调用run()**：

```java
public class MyThread extends Thread {
	@Override
	public void run() {
		// TODO Auto-generated method stub
		super.run();
		System.out.println("MyThread");
		System.out.println(Thread.currentThread().getName());
	}
}
public class Test {

	public static void main(String[] args) {
		MyThread mThread = new MyThread();
		System.out.println("--" + Thread.currentThread().getName());
		mThread.run();
		System.out.println("end!!!");
	}
}

输出：
--main
MyThread
main
end!!!
```

# this、Thread.currentThread()和this.currentThread()的区别
参见[Java 多线程之 this 与 Thread.currentThread() 的区别](https://www.jianshu.com/p/4fe824ffbeac)
- this指的是当前对象，而Thread.currentThread()指的是当前运行的线程对象，这两者有时不一定相同。
- Thread.currentThread()和this.currentThread()在大部分情况下结果是相同的，都是调用Thread的静态方法currentThread()，若当前子类重写了父类的currentThread()方法，那么Thread.currentThread()和this.currentThread()返回的结果会不同。


