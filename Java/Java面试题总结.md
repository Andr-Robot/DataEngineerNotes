* [==、equal和hashcode区别](#equal和hashcode区别)
    * [关系操作符 ==](#关系操作符-)
    * [equals方法](#equals方法)
    * [hashCode方法](#hashcode方法)
    * [equals 与 hashCode](#equals-与-hashcode)
    * [Hashset、Hashmap、Hashtable与hashcode()和equals()的密切关系](#hashsethashmaphashtable与hashcode和equals的密切关系)
    * [参考文献](#参考文献)
* [treemap和hashmap的区别](#treemap和hashmap的区别)
    * [HashMap](#hashmap)
    * [TreeMap](#treemap)
    * [总结](#总结)
    * [参考文献](#参考文献)
* [java里set，Array和map等容器了解吗，它们的继承关系写一下。](#java里setarray和map等容器了解吗它们的继承关系写一下)
* [为什么wait和notify在Object方法上？](#为什么wait和notify在object方法上)
* [String、StringBuffer、StringBuilder的区别](#stringstringbufferstringbuilder的区别)
* [什么时候进行FullGC？](#什么时候进行fullgc)
* [什么是线程安全，怎么保证多线程线程安全？](#什么是线程安全怎么保证多线程线程安全)
* [为什么使用线程池？线程池的好处？](#为什么使用线程池线程池的好处)
* [什么是内部类，什么是匿名内部类？](#什么是内部类什么是匿名内部类)
    * [成员内部类](#成员内部类)
    * [匿名内部类](#匿名内部类)
    * [静态内部类](#静态内部类)
    * [局部内部类](#局部内部类)
* [Java堆溢出问题怎么处理，内存泄漏和内存溢出的区别](#java堆溢出问题怎么处理内存泄漏和内存溢出的区别)
* [jvm配置-Xms和-Xmx分别指什么](#jvm配置-xms和-xmx分别指什么)

# ==、equal和hashcode区别
- ==： 该操作符生成的是一个boolean结果，它计算的是操作数的值之间的关系
- equals： Object 的 实例方法，比较两个对象的内容是否相同
- hashCode： Object 的 native方法 , 获取对象的哈希值，用于确定该对象在哈希表中的索引位置，它实际上是一个int型整数

## 关系操作符 ==
1. 若操作数的类型是**基本数据类型**，则该关系操作符判断的是左右两边操作数的**值**是否相等
2. 若操作数的类型是**引用数据类型**，则该关系操作符判断的是左右两边操作数的**内存地址**是否相同。也就是说，若此时返回true,则该操作符作用的一定是同一个对象。   

## equals方法
比较两个对象的 content 是否相同。   
equals方法，内部实现分为三个步骤：
- 先 比较引用是否相同(是否为同一对象),
- 再 判断类型是否一致（是否为同一类型）,
- 最后 比较内容是否一致

Java 中所有内置的类的 equals 方法的实现步骤均是如此，特别是诸如 Integer，Double 等包装器类。

## hashCode方法
由 Object 类定义的 hashCode 方法会针对**不同的对象返回不同的整数**。

## equals 与 hashCode
Java对象的eqauls方法和hashCode方法是这样规定的：
1. 相等（相同）的对象必须具有相等的哈希码（或者散列码）。
2. 如果两个对象的hashCode相同，它们并不一定相同。

## Hashset、Hashmap、Hashtable与hashcode()和equals()的密切关系
H**ashset是继承Set接口，Set接口又实现Collection接口，这是层次关系。那么Hashset、Hashmap、Hashtable中的存储操作是根据什么原理来存取对象的呢？**   
下面以HashSet为例进行分析，我们都知道：在**hashset中不允许出现重复对象**，元素的位置也是不确定的。在hashset中又是怎样判定元素是否重复的呢？在java的集合中，判断两个对象是否相等的规则是：
1. 判断两个对象的hashCode是否相等
    - 如果不相等，认为两个对象也不相等，完毕
    - 如果相等，转入2
（这一点只是为了提高存储效率而要求的，其实理论上没有也可以，但如果没有，实际使用时效率会大大降低，所以我们这里将其做为必需的。）
2. 判断两个对象用equals运算是否相等
    - 如果不相等，认为两个对象也不相等
    - 如果相等，认为两个对象相等（equals()是判断两个对象是否相等的关键）

**为什么是两条准则，难道用第一条不行吗？**  
不行，因为前面已经说了，hashcode()相等时，equals()方法也可能不等，所以必须用第2条准则进行限制，才能保证加入的为非重复元素。

## 参考文献
[Java 中的 ==, equals 与 hashCode 的区别与联系](https://blog.csdn.net/justloveyou_/article/details/52464440)    
[Java hashCode() 和 equals()的若干问题解答](http://www.cnblogs.com/skywang12345/p/3324958.html)   
[Java提高篇——equals()与hashCode()方法详解](https://www.cnblogs.com/Qian123/p/5703507.html#_label1)

# treemap和hashmap的区别
## HashMap
HashMap最多只允许一条Entry的键为Null(多条会覆盖)，但允许多条Entry的值为Null。默**认初始容量 (16) 和 默认负载因子(0.75)**。扩容直接是原来的2倍。

HashMap不支持线程的同步。

HashMap散列图、Hashtable散列表是按“有利于随机查找的散列(hash)的顺序”。**并非按输入顺序。遍历时只能全部输出，而没有顺序**。

## TreeMap
TreeMap继承自SortedMap（添加到SortedMap实现类的元素**必须实现Comparable接口**，否则您必须**给它的构造函数提供一个Comparator接口的实现**）。TreeMap能够**把它保存的记录根据键排序**，默认是**按升序排序**，也**可以指定排序的比较器**。当用Iteraor遍历TreeMap时，得到的记录是排过序的。**TreeMap的键和值都不能为空**。

TreeMap取出来的是排序后的键值对。但如果您要按**自然顺序或自定义顺序遍历键**，那么TreeMap会更好。

TreeMap基于**红黑树**（Red-Black tree）实现。该映射根据其键的**自然顺序**进行排序，或者根据创建映射时提供的 **Comparator** 进行排序，**具体取决于使用的构造方法**。TreeMap的基本操作 containsKey、get、put 和 remove 的时间复杂度是**log(n)**。另外，TreeMap是**非同步**的。 它的iterator 方法返回的迭代器是fail-fastl的。

```java
// 默认构造函数。使用该构造函数，TreeMap中的元素按照自然排序进行排列。
public TreeMap() {
    comparator = null;
}

// 构造一个新的、空的树映射，该映射根据给定比较器进行排序。
public TreeMap(Comparator<? super K> comparator) {
    this.comparator = comparator;
}

// 构造一个与给定映射具有相同映射关系的新的树映射，该映射根据其键的自然顺序 进行排序。
public TreeMap(Map<? extends K, ? extends V> m) {
    comparator = null;
    putAll(m);
}

// 构造一个与指定有序映射具有相同映射关系和相同排序顺序的新的树映射。
public TreeMap(SortedMap<K, ? extends V> m) {
    comparator = m.comparator();
    try {
        buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
    } catch (java.io.IOException cannotHappen) {
    } catch (ClassNotFoundException cannotHappen) {
    }
}
```

## 总结
1. TreeMap是根据key进行排序的，它的排序和定位需要依赖比较器或覆写Comparable接口，也因此不需要key覆写hashCode方法和equals方法，就可以排除掉重复的key，而HashMap的key则需要通过覆写hashCode方法和equals方法来确保没有重复的key。
2. TreeMap的查询、插入、删除效率均没有HashMap高，一般只有要对key排序时才使用TreeMap。
3. TreeMap的key不能为null，而HashMap的key可以为null。
4. TreeMap不是同步的。如果多个线程同时访问一个映射，并且其中至少一个线程从结构上修改了该映射，则其必须 外部同步。

    ```
    SortedMap m = Collections.synchronizedSortedMap(new TreeMap(...));
    ```

## 参考文献
[java集合系列——Map之TreeMap介绍（九）](https://blog.csdn.net/u010648555/article/details/60476232)    
[Java 集合系列12之 TreeMap详细介绍(源码解析)和使用示例](https://www.cnblogs.com/skywang12345/p/3310928.html)    

# java里set，Array和map等容器了解吗，它们的继承关系写一下。


# 为什么wait和notify在Object方法上？
wait()和notify()是Java给我们提供线程之间通信的API。因为我们的锁是对象锁，每个对象都可以成为锁。让当前线程等待某个对象的锁，当然应该通过这个对象来操作了。而锁对象是任意的，所以这些方法必须定义在Object类中。

# String、StringBuffer、StringBuilder的区别
- String 字符串常量
- StringBuffer 字符串变量（线程安全）
- StringBuilder 字符串变量（非线程安全）

**String 类型和 StringBuffer 类型的主要性能区别其实在于 String 是不可变的对象,** 因此在每次对 String 类型进行改变的时候其实都等同于生成了一个新的 String 对象，然后将指针指向新的 String 对象，所以经常改变内容的字符串最好不要用 String ，因为每次生成对象都会对系统性能产生影响，特别当内存中无引用对象多了以后， JVM 的 GC 就会开始工作，那速度是一定会相当慢的。

而如果是使用 StringBuffer 类则结果就不一样了，**每次结果都会对 StringBuffer 对象本身进行操作**，而不是生成新的对象，再改变对象引用。

> StringBuilder > StringBuffer > String

# 什么时候进行FullGC？
- System.gc()方法的调用，此方法的调用是建议JVM进行Full GC,虽然只是建议而非一定。
- 老年代代空间不足

# 什么是线程安全，怎么保证多线程线程安全？
如果一个对象在多线程环境下是安全的，那么多个线程在调用这个对象时，不需要要任何额外的手段就能在多次重复执行的情况得到一致且正确的结果。

线程安全是**通过线程同步控制来实现的**，也就是synchronized关键字。

# 为什么使用线程池？线程池的好处？
如果使用`new Thread(...).start()`的方法处理多线程，有如下缺点：
- **开销大**。对于JVM来说，每次新建线程和销毁线程都会有很大的开销。
- **线程缺乏管理**。没有一个池来限制线程的数量，如果并发量很高，会创建很多的线程，而且线程之间可能会有相互竞争，这将会过多得占用系统资源，增加系统资源的消耗量。而且线程数量超过系统负荷，容易导致系统不稳定。

使用线程池的方式，有如下优点：
- **复用线程**。通过复用创建的了的线程，减少了线程的创建、消亡的开销。
- **有效控制并发线程数**。
- **提供了更简单灵活的线程管理**。可以提供定时执行、定期执行、单线程、可变线程数等多种线程使用功能。

# 什么是内部类，什么是匿名内部类？
参见[Java内部类和匿名内部类的用法](https://blog.csdn.net/guyuealian/article/details/51981163)和[Java内部类详解](https://www.cnblogs.com/dolphin0520/p/3811445.html)

在Java中，可以**将一个类定义在另一个类里面或者一个方法里面**，这样的类称为内部类。广泛意义上的内部类一般来说包括这四种：**成员内部类、局部内部类、匿名内部类和静态内部类**。

## 成员内部类
它的定义位于另一个类的内部。成员内部类可以无条件访问外部类的所有成员属性和成员方法（包括private成员和静态成员）。在外部类中如果要访问成员内部类的成员，必须先创建一个成员内部类的对象，再通过指向这个对象的引用来访问。**成员内部类是依附外部类而存在的**，也就是说，如果要创建成员内部类的对象，前提是必须存在一个外部类的对象。

## 匿名内部类
匿名内部类是唯一一种没有构造器的类。大部分匿名内部类用于接口回调。匿名内部类在编译的时候由系统自动起名为`Outter$1.class`。一般来说，匿名内部类用于继承其他类或是实现接口，并不需要增加额外的方法，只是对继承方法的实现或是重写。

## 静态内部类
静态内部类也是定义在另一个类里面的类，只不过在类的前面多了一个关键字static。**静态内部类是不需要依赖于外部类的**，这点和类的静态成员属性有点类似，并且它不能使用外部类的非static成员变量或者方法。

## 局部内部类
局部内部类是定义在一个方法或者一个作用域里面的类，它和成员内部类的区别在于局部内部类的访问仅限于方法内或者该作用域内。局部内部类就像是方法里面的一个局部变量一样，是不能有public、protected、private以及static修饰符的。

# Java堆溢出问题怎么处理，内存泄漏和内存溢出的区别
- **内存溢出 out of memory**：是指程序在申请内存时，没有足够的内存空间供其使用，出现out of memory；比如申请了一个integer,但给它存了long才能存下的数，那就是内存溢出。
- **内存泄露 memory leak**：是指程序在申请内存后，无法释放已申请的内存空间，一次内存泄露危害可以忽略，但内存泄露堆积后果很严重，无论多少内存,迟早会被占光。

memory leak会最终会导致out of memory。

内存泄露的几种场景：
- 长生命周期的对象持有短生命周期对象的引用

内存溢出的几种情况：
- 堆内存溢出（outOfMemoryError：java heap space），堆中的内存是用来生成对象实例和数组的。
- 方法区内存溢出（outOfMemoryError：permgem space），方法区主要存放的是类信息、常量、静态变量等。如果程序加载的类过多就可能导致该区域发生内存溢出。
- 虚拟机栈栈溢出（java.lang.StackOverflowError），一般虚拟机栈溢出是由于**递归太深或方法调用层级过多**导致的。

为了避免内存泄露，在编写代码的过程中可以参考下面的建议：
1. 尽早释放无用对象的引用
2. 使用字符串处理，避免使用String，应大量使用StringBuffer，每一个String对象都得独立占用内存一块区域
3. 尽量少用静态变量，因为静态变量存放在永久代（方法区），永久代基本不参与垃圾回收
4. 避免在循环中创建对象
5. 开启大型文件或从数据库一次拿了太多的数据很容易造成内存溢出，所以在这些地方要大概计算一下数据量的最大值是多少，并且设定所需最小及最大的内存空间值。

# jvm配置-Xms和-Xmx分别指什么
- -Xms：JVM初始内存大小
- -Xmx：JVM最大可用内存大小




