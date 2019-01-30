[toc]

锁——是为了**解决并发操作引起的脏读、数据不一致的问题**。JVM提供了`synchronized`关键字来实现对变量的同步访问以及用`wait`和`notify`来实现线程间通信。在jdk1.5以后，JAVA提供了`Lock`类来实现和`synchronized`一样的功能，并且还提供了`Condition`来显示线程间通信。

# synchronized的缺陷
## 影响执行效率
如果一个代码块被synchronized修饰了，当一个线程获取了对应的锁，并执行该代码块时，**其他线程便只能一直等待**，等待获取锁的线程释放锁，而这里获取锁的线程释放锁只会有两种情况：
1. 获取锁的线程执行完了该代码块，然后线程释放对锁的占有；
2. 线程执行发生异常，此时JVM会让线程自动释放锁。

那么如果这个获取锁的线程由于要等待IO或者其他原因（比如调用sleep方法）被阻塞了，但是又没有释放锁，其他线程便只能干巴巴地等待，这样影响程序执行效率。

因此就需要有一种机制可以不让等待的线程一直无期限地等待下去（比如只等待一定的时间或者能够响应中断），通过Lock就可以办到。

## 无法更好的处理读读操作
当有多个线程读写文件时，读操作和写操作会发生冲突现象，写操作和写操作会发生冲突现象，但是读操作和读操作不会发生冲突现象。

但是采用synchronized关键字来实现同步的话，就会导致一个问题：    
- 如果多个线程都只是进行读操作，所以当一个线程在进行读操作时，其他线程只能等待无法进行读操作。

因此就需要一种机制来使得多个线程都只是进行读操作时，线程之间不会发生冲突，通过Lock就可以办到。

## 无法感知是否成功获得锁
通过Lock可以知道线程有没有成功获取到锁。这个是synchronized无法办到的。

## 总结
也就是说**Lock提供了比synchronized更多的功能**。但是要注意以下几点：
1. Lock不是Java语言内置的，synchronized是Java语言的关键字，因此是内置特性。Lock是一个类，通过这个类可以实现同步访问；
2. Lock和synchronized有一点非常大的不同，**采用synchronized不需要用户去手动释放锁**，当synchronized方法或者synchronized代码块执行完之后，系统会自动让线程释放对锁的占用；而**Lock则必须要用户去手动释放锁**，如果没有主动释放锁，就有可能导致出现死锁现象。

# Lock
java.util.concurrent.locks包中的接口。

```java
public interface Lock {

    void lock();

    void lockInterruptibly() throws InterruptedException;

    boolean tryLock();
    
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

    void unlock();
    
    Condition newCondition();
}
```

其中lock()、tryLock()、tryLock(long time, TimeUnit unit)和lockInterruptibly()是用来获取锁的。unLock()方法是用来释放锁的。newCondition()用于线程协作。

## lock()
用来获取锁。如果锁已被其他线程获取，则进行等待。

如果**采用Lock，必须主动去释放锁，并且在发生异常时，不会自动释放锁**。所以使用Lock必须在`try{}catch{}`块中进行，并且**将释放锁的操作放在`finally`块中进行**，以保证锁一定被被释放，防止死锁的发生。通常使用Lock来进行同步的话，是以下面这种形式去使用的：

```java
Lock lock = ...;
lock.lock();
try{
    //处理任务
}catch(Exception ex){
     
}finally{
    lock.unlock();   //释放锁
}
```

## tryLock()
`tryLock()`方法是**有返回值**的，它表示用来尝试获取锁，**如果获取成功，则返回true，如果获取失败（即锁已被其他线程获取），则返回false**，也就说这个方法无论如何都会立即返回。在拿不到锁时不会一直在那等待。

```java
Lock lock = ...;
if(lock.tryLock()) {
     try{
         //处理任务
     }catch(Exception ex){
         
     }finally{
         lock.unlock();   //释放锁
     } 
}else {
    //如果不能获取锁，则直接做其他事情
}
```

## tryLock(long time, TimeUnit unit)
`tryLock(long time, TimeUnit unit)`方法和`tryLock()`方法是类似的，只不过区别在于这个方法**在拿不到锁时会等待一定的时间**，在时间期限之内如果还拿不到锁，就返回false。如果如果一开始拿到锁或者在等待期间内拿到了锁，则返回true。

## lockInterruptibly()
`lockInterruptibly()`方法比较特殊，当通过这个方法去获取锁时，**如果线程正在等待获取锁，则这个线程能够响应中断，即中断线程的等待状态**。也就使说，当两个线程同时通过`lock.lockInterruptibly()`想获取某个锁时，假若此时线程A获取到了锁，而线程B只有在等待，那么对线程B调用`threadB.interrupt()`方法能够中断线程B的等待过程。

由于`lockInterruptibly()`的声明中抛出了异常，所以`lock.lockInterruptibly()`必须放在`try`块中或者在调用`lockInterruptibly()`的方法外声明抛出`InterruptedException`。

因此`lockInterruptibly()`一般的使用形式如下：

```java
public void method() throws InterruptedException {
    lock.lockInterruptibly();
    try {  
     //.....
    }
    finally {
        lock.unlock();
    }  
}
```

**注意：** 
- 当一个线程获取了锁之后，是不会被`interrupt()`方法中断的。因为单独调用`interrupt()`方法不能中断正在运行过程中的线程，只能中断阻塞过程中的线程。
- 因此当通过`lockInterruptibly()`方法获取某个锁时，如果不能获取到，只有进行等待的情况下，是可以响应中断的。
- 而用`synchronized`修饰的话，当一个线程处于等待某个锁的状态，是无法被中断的，只有一直等待下去。

# ReentrantLock
ReentrantLock，意思是“**可重入锁**”。**ReentrantLock是唯一实现了Lock接口的类**，并且ReentrantLock提供了更多的方法。

ReentrantLock可以通过调用不同的构造方法，生成公平锁或不公平锁。

```java
/**
 * Creates an instance of {@code ReentrantLock}.
 * This is equivalent to using {@code ReentrantLock(false)}.
 */
public ReentrantLock() {
    sync = new NonfairSync();
}

/**
 * Creates an instance of {@code ReentrantLock} with the
 * given fairness policy.
 *
 * @param fair {@code true} if this lock should use a fair ordering policy
 */
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

# ReadWriteLock
ReadWriteLock也是一个接口，在它里面只定义了两个方法：

```java
public interface ReadWriteLock {

    Lock readLock();

    Lock writeLock();
}
```

一个用来获取读锁，一个用来获取写锁。也就是说将文件的读写操作分开，分成2个锁来分配给线程，从而使得多个线程可以同时进行读操作。下面的ReentrantReadWriteLock实现了ReadWriteLock接口。

# ReentrantReadWriteLock
ReentrantReadWriteLock里面提供了很多丰富的方法，不过最主要的有两个方法：`readLock()`和`writeLock()`用来获取读锁和写锁。

使用synchronized实现多线程的读操作，会出现每个线程同步完成的情况。

```java
public class Test {
    public static void main(String[] args)  {
        final Test test = new Test();
         
        new Thread(){
            public void run() {
                test.get(Thread.currentThread());
            };
        }.start();
         
        new Thread(){
            public void run() {
                test.get(Thread.currentThread());
            };
        }.start();
         
    }  
     
    public synchronized void get(Thread thread) {
    	int i = 0;
        while(i < 8) {
            System.out.println(thread.getName()+"正在进行读操作");
            i += 1;
        }
        System.out.println(thread.getName()+"读操作完毕");
    }
}

输出：
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0读操作完毕
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1读操作完毕
```

当使用ReentrantReadWriteLock时。

```java
public class Test {
	private ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
	
    public static void main(String[] args)  {
        final Test test = new Test();
         
        new Thread(){
            public void run() {
                test.get(Thread.currentThread());
            };
        }.start();
         
        new Thread(){
            public void run() {
                test.get(Thread.currentThread());
            };
        }.start();
         
    }  
     
    public void get(Thread thread) {
    	lock.readLock().lock();
    	try {
    		int i = 0;
            while(i < 1000) {
                System.out.println(thread.getName()+"正在进行读操作");
                i += 1;
            }
            System.out.println(thread.getName()+"读操作完毕");
		} finally {
			lock.readLock().unlock();
		}
    }
}

输出：
...
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-0正在进行读操作
Thread-1正在进行读操作
Thread-1正在进行读操作
...
```

可以看到两个线程是异步进行读操作的，这样就大大提升了读操作的效率。

**注意：** 
- 如果有一个线程已经占用了读锁，则此时其他线程如果要申请写锁，则申请写锁的线程会一直等待释放读锁。
- 如果有一个线程已经占用了写锁，则此时其他线程如果申请写锁或者读锁，则申请的线程会一直等待释放写锁。

# 锁的相关概念
## 公平锁/非公平锁
- 公平锁是指多个线程按照申请锁的顺序来获取锁。比如同时有多个线程在等待一个锁，当这个锁被释放时，**等待时间最久的线程**（最先请求的线程）会获得该所，这种就是公平锁。
- 非公平锁是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。这样就可能导致某个或者一些线程永远获取不到锁。

对于Java ReentrantLock而言，通过构造函数指定该锁是否是公平锁，默认是非公平锁。**非公平锁的优点在于吞吐量比公平锁大**。对于Synchronized而言，也是一种非公平锁。由于其并不像ReentrantLock是通过AQS的来实现线程调度，所以并没有任何办法使其变成公平锁。

## 乐观锁/悲观锁
悲观锁认为对于同一个数据的并发操作，一定是会发生修改的，哪怕没有修改，也会认为发生了修改。因此对于同一个数据的并发操作，悲观锁采取加锁的形式。悲观的认为，不加锁的并发操作一定会出问题。    

乐观锁则认为对于同一个数据的并发操作，是不会发生修改的。在更新数据的时候需要判断该数据是否被别人修改过。如果数据被其他线程修改，则不进行数据更新，如果数据没有被其他线程修改，则进行数据更新。乐观的认为，不加锁的并发操作是没有问题的。

- 悲观锁：**假定会发生并发冲突**，屏蔽一切可能违反数据完整性的操作。 
- 乐观锁：**假定不会发生并发冲突**，只在提交操作时检测是否违反数据完整性（使用版本号或者时间戳来配合实现）。乐观锁不能解决脏读的问题。

**适用场景：**
- 悲观锁：比较适合写入操作比较频繁的场景，如果出现大量的读取操作，每次读取的时候都会进行加锁，这样会增加大量的锁的开销，降低了系统的吞吐量。
- 乐观锁：比较适合读取操作比较频繁的场景，如果出现大量的写入操作，数据发生冲突的可能性就会增大，为了保证数据的一致性，应用层需要不断的重新获取数据，这样会增加大量的查询操作，降低了系统的吞吐量。

**总结：** 两种所各有优缺点，**写入频繁使用悲观锁，读取频繁使用乐观锁**。

## 独享锁/共享锁
- 独享锁是指该锁一次只能被一个线程所持有。获得独享锁的事务即能读数据又能修改数据。
- 共享锁是指该锁可被多个线程所持有。获准共享锁的事务只能读数据，不能修改数据。 
 
对于Java **ReentrantLock**而言，其是独享锁。但是对于Lock的另一个实现类**ReentrantReadWriteLock**，其读锁是共享锁，其写锁是独享锁。读锁的共享锁可保证并发读是非常高效的。但是读写，写读，写写的过程是互斥的。独享锁与共享锁也是通过AQS来实现的，通过实现不同的方法，来实现独享或者共享。对于**Synchronized**而言，当然是独享锁。

## 互斥锁/读写锁
上面讲的独享锁/共享锁就是一种广义的说法，互斥锁/读写锁就是具体的实现。
- 所谓互斥锁就是指一次最多只能有一个线程持有的锁。在JDK中synchronized和JUC的Lock就是互斥锁。
- 读写锁是一个资源能够被多个读线程访问，或者被一个写线程访问但不能同时存在读线程。Java当中的读写锁通过ReentrantReadWriteLock实现。

## 可重入锁
可重入锁又名**递归锁**，是指在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。对于Java `ReentrantLock`而言,他的名字就可以看出是一个可重入锁。对于`Synchronized`而言,也是一个可重入锁。**可重入锁的一个好处是可一定程度避免死锁**。



# 参考文献
[Java中的锁[原理、锁优化、CAS、AQS]](https://www.jianshu.com/p/e674ee68fd3f)    
[[Java并发系列] 3.Java中的锁](https://juejin.im/post/5a0ac89df265da43247ff78e)    
[Java中的锁以及sychronized实现机制](https://segmentfault.com/a/1190000013512810)    
[Java中的锁](https://blog.csdn.net/u013256816/article/details/51204385)

