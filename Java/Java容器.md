[toc]

容器主要包括 Collection 和 Map 两种，Collection 又包含了 List、Set 以及 Queue。详情参见
[Java 容器——Interview-Notebook](https://github.com/CyC2018/Interview-Notebook/blob/master/notes/Java%20%E5%AE%B9%E5%99%A8.md#map)。

# HashSet
HashSet是基于HashMap来实现的，**其元素值可以是NULL**。**它不能保证元素的排列顺序**。同样，HashSet是不同步的。

# HashTable
参见[Map 综述（四）：彻头彻尾理解 HashTable](https://blog.csdn.net/justloveyou_/article/details/72862373)    
底层实现都是一个**链表数组**，具有**寻址容易、插入和删除也容易**的特性。**默认初始容量(11)和默认负载因子(0.75f)**。调整Hashtable的长度，将长度变成原来的(2倍+1)。

```java
int newCapacity = (oldCapacity << 1) + 1;
```

# HashMap
参考[Java HashMap工作原理及实现](https://yikun.github.io/2015/04/01/Java-HashMap%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)、[Java：手把手带你源码分析 HashMap 1.7](https://blog.csdn.net/carson_ho/article/details/79373026)、[Map 综述（一）：彻头彻尾理解 HashMap](https://blog.csdn.net/justloveyou_/article/details/62893086)和[Java 集合系列10之 HashMap详细介绍(源码解析)和使用示例](http://www.cnblogs.com/skywang12345/p/3310835.html)。   

HashMap最多只允许一条Entry的键为Null(多条会覆盖)，但允许多条Entry的值为Null。默**认初始容量 (16) 和 默认负载因子(0.75)**。扩容直接是原来的2倍。

```
newCap = oldCap << 1
```

HashMap | HashTable
---|---
继承的是AbstractMap类 | 继承的的是Dicionary类
非线程安全（效率比较的高） | 线程安全（效率相对比较低）
允许存在null的key | 不允许存在空key

HashMap 的底层数组长度总是2的n次方的原因有两个，即当 length=2^n 时：
1. 不同的hash值发生碰撞的概率比较小，这样就会使得数据在table数组中分布较均匀，空间利用率较高，查询速度也较快；
2. 对于取余运算，我们首先想到的是：**哈希值%length = bucketIndex**。但是在HashMap中是通过`h&(length - 1)` 就相当于对length取模，而且在速度、效率上比直接取模要快得多，即二者是等价不等效的，这是HashMap在速度和效率上的一个优化。

# ConcurrentHashMap
参考[Java7/8 中的 HashMap 和 ConcurrentHashMap 全解析](https://javadoop.com/post/hashmap)和[Map 综述（三）：彻头彻尾理解 ConcurrentHashMap](https://blog.csdn.net/justloveyou_/article/details/72783008)

在理想状态下，ConcurrentHashMap 可以支持 16 个线程执行并发写操作（如果并发级别设为16），及任意数量线程的读操作。**默认初始容量(16)、默认负载因子(0.75)和默认并发级别(16)**。       

ConcurrentHashMap本质上是**一个Segment数组，而一个Segment实例又包含若干个桶，每个桶中都包含一条由若干个 HashEntry 对象链接起来的链表**。总的来说，ConcurrentHashMap的高效并发机制是通过以下三方面来保证的：
- 通过锁分段技术保证并发环境下的写操作；
- 通过 HashEntry的不变性、Volatile变量的内存可见性和加锁重读机制保证高效、安全的读操作；
- 通过不加锁和加锁两种方案控制跨段操作的的安全性。



# LinkedHashMap
参考[Java LinkedHashMap工作原理及实现](https://yikun.github.io/2015/04/02/Java-LinkedHashMap%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/) 和[Map 综述（二）：彻头彻尾理解 LinkedHashMap](https://blog.csdn.net/justloveyou_/article/details/71713781)   

LinkedHashMap可以认为是HashMap+LinkedList，即它既使用HashMap操作数据结构，又使用LinkedList维护插入元素的先后顺序

- 有序的Map接口实现，有序指元素**可以按照访问顺序或插入顺序迭代**。内部维护了一个**双向循环链表**来实现有序。
- **非线程安全**。

它通过维护一个额外的**双向链表**保证了迭代顺序。特别地，该迭代顺序**可以是插入顺序，也可以是访问顺序**。因此，根据链表中元素的顺序可以将LinkedHashMap分为：保持插入顺序的LinkedHashMap 和 保持访问顺序的LinkedHashMap，其中LinkedHashMap的默认实现是按插入顺序排序的。

# Vector、ArrayList和Linkedlist的区别
- **ArrayList**：采用的是**数组**形式来保存对象的，可以认为ArrayList是一个可改变大小的数组。随着越来越多的元素被添加到ArrayList中，其规模是动态增加的。这种方式将对象放在连续的位置中，所以最大的缺点就是插入删除时非常麻烦，优点是查询速度快。
    - ArrayList **初始化大小是 10**；
    - **扩容点规则**：新增的时候发现容量不够用了，就去扩容； 
    - **扩容大小规则**：扩容后的大小= 原始大小+原始大小/2 + 1。（例如：原始大小是 10 ，扩容后的大小就是 10 + 5+1 = 16）
    - get()方法的时间复杂度是O(1)，对于随机位置的add/remove，时间复杂度为 O(n),但是对于列表末尾的添加/删除操作,时间复杂度是 O(1).
- **LinkedList**：底层是通过**双向链表**实现的，采用的将对象存放在独立的空间中，而且在每个空间中还保存下一个链接的索引，但是缺点就是查找非常慢，要从第一个索引开始，优点是插入删除快。
    - **没有初始化大小，也没有扩容的机制**，就是一直在前面或者后面新增就好。
    - get()方法的时间复杂度是O(n)，对于随机位置的add/remove，时间复杂度为 O(n),但是对于列表 末尾/开头 的添加/删除操作，时间复杂度是 O(1)。
- **Vector**：Vector和ArrayList一样，都是通过**数组**实现的，但是**Vector是线程安全的**，其中的很多方法都通过同步（synchronized）处理来保证线程安全。ArrayList和Linkedlist这两个都是非线程安全的。
    - Vector默认**初始容量为10**；
    - **扩容增量**：原容量的 1倍，如 Vector的容量为10，一次扩容后是容量为20。

**总结：**
- 对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。
- 对于新增和删除操作add和remove，LinkedList比较占优势，因为ArrayList要移动数据。若只对单条数据插入或删除，ArrayList的速度反而优于LinkedList。但若是批量随机的插入删除数据，LinkedList的速度大大优于ArrayList. 因为ArrayList每插入一条数据，要移动插入点及之后的所有数据。

# HashTable和HashMap的区别
Hashtable与HashMap还是有一定的差别的：
- Hashtable不同于HashMap，前者既不允许key为null，也不允许value为null;
- HashMap中用于定位桶位的Key的hash的计算过程要比Hashtable复杂一点，没有Hashtable如此简单、直接（Hashtable只需通过key的hash值定位到table数组的某个特定的桶）；
- 在HashMap的插入K/V对的过程中，总是先插入后检查是否需要扩容；而Hashtable则是先检查是否需要扩容后插入；
- Hashtable的方法是**同步**的，实现**线程安全**的Map；而HashMap的方法不是同步的，是Map的非线程安全实现。

# 参考文献
[HashMap、LinkedHashMap、ConcurrentHashMap、ArrayList、LinkedList的底层实现](https://blog.csdn.net/baidu_28068985/article/details/78529246)