ConcurrentHashMap本质上**是一个Segment数组，而一个Segment实例又包含若干个桶，每个桶中都包含一条由若干个 HashEntry 对象链接起来的链表**。总的来说，ConcurrentHashMap的高效并发机制是通过以下三方面来保证的：
- 通过锁分段技术保证并发环境下的写操作；
- 通过 HashEntry的不变性、Volatile变量的内存可见性和加锁重读机制保证高效、安全的读操作；
- 通过不加锁和加锁两种方案控制跨段操作的的安全性。

# 多线程下HashMap
HashMap在并发环境下使用中最为典型的一个问题，就是**在HashMap进行扩容重哈希时导致Entry链形成环**。一旦Entry链中有环，势必会导致在同一个桶中进行插入、查询、删除等操作时陷入死循环。

# ConcurrentHashMap定义
**ConcurrentHashMap不允许Key和Value为null**。

ConcurrentHashMap类中包含两个静态内部类 **HashEntry** 和 **Segment**。
- HashEntry 用来封装具体的K/V对，是个典型的四元组；
- Segment 用来充当锁的角色，每个 Segment 对象守护整个ConcurrentHashMap的若干个桶 (可以把Segment看作是一个小型的哈希表)，其中每个桶是由若干个 HashEntry 对象链接起来的链表。

ConcurrentHashMap 是**一个 Segment 数组，Segment 通过继承 ReentrantLock 来进行加锁**，所以每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全。

ConcurrentHashMap **默认有 16 个** Segments，所以理论上，这个时候，最多可以同时支持 16 个线程并发写，只要它们的操作分别分布在不同的 Segment 上。这个值可以在初始化的时候设置为其他值，但是**一旦初始化以后，它是不可以扩容的**。   
## 初始化
- **initialCapacity**：初始容量，这个值指的是整个 ConcurrentHashMap 的初始容量，实际操作的时候需要平均分给每个 Segment。
- **loadFactor**：负载因子，之前我们说了，Segment 数组不可以扩容，所以这个负载因子是给每个 Segment 内部使用的。

用 new ConcurrentHashMap() 无参构造函数进行初始化的，那么初始化完成后：
- Segment 数组长度为 16，不可以扩容；
- Segment[i] 的默认大小为 2，负载因子是 0.75，得出初始阈值为 1.5，也就是以后插入第一个元素不会触发扩容，插入第二个会进行第一次扩容；
- 这里初始化了 segment[0]，其他位置还是 null；
- 当前 segmentShift 的值为 32 - 4 = 28，segmentMask 为 16 - 1 = 15，姑且把它们简单翻译为**移位数和掩码**。

## Put过程
1. 计算 key 的 hash 值，根据 hash 值找到 Segment 数组中的位置 j
2. 如果当前segment是插入第一个值，则使用`ensureSegment(int k)`来对当前segment进行初始化。（使用当前`segment[0]`的长度和负载因子来初始化）
3. 插入新值到 槽 Segment 中。在往该 segment 写入前，需要先获取该 segment 的独占锁。
4. tryLock()失败后执行scanAndLockForPut方法，循环申请锁，如果申请到锁则返回，如果超过最大自旋次数则通过lock()加锁，进入阻塞队列。
5. 获取锁之后，去判断当前<key, value>中key是否存在，如果存在就更改旧值，如果不存在则插入新值，使用头插法。
6. 当插入新值时，如果该segment中的链表的长度超过阈值，则进行扩容和重哈希操作。**重哈希时不需要考虑并发，因为到这里的时候，是持有该 segment 的独占锁的**。

## Get过程
1. 计算 hash 值，找到 segment 数组中的具体位置，或我们前面用的“槽”
2. 槽中也是一个数组，根据 hash 找到数组中具体的位置
3. 到这里是链表了，顺着链表进行查找即可

## 并发问题分析
**添加节点的操作 put 和删除节点的操作 remove 都是要加 segment 上的独占锁的**，所以它们之间自然不会有问题，我们需要考虑的问题就是 get 的时候在同一个 segment 中发生了 put 或 remove 操作。

### put 操作的线程安全性
1. 初始化槽，这个我们之前就说过了，使用了 CAS 来初始化 Segment 中的数组。
2. 添加节点到链表的操作是插入到表头的，所以，如果这个时候 get 操作在链表遍历的过程已经到了中间，是不会影响的。当然，另一个并发问题就是 get 操作在 put 之后，需要保证刚刚插入表头的节点被读取，这个依赖于 setEntryAt 方法中使用的 UNSAFE.putOrderedObject。
3. 扩容。扩容是新创建了数组，然后进行迁移数据，最后面将 newTable 设置给属性 table。所以，如果 get 操作此时也在进行，那么也没关系，如果 get 先行，那么就是在旧的 table 上做查询操作；而 put 先行，那么 put 操作的可见性保证就是 table 使用了 volatile 关键字。

### remove 操作的线程安全性
get 操作需要遍历链表，但是 remove 操作会"破坏"链表。

如果 remove 破坏的节点 get 操作已经过去了，那么这里不存在任何问题。

如果 remove 先破坏了一个节点，分两种情况考虑。   
1. 如果此节点是头结点，那么需要将头结点的 next 设置为数组该位置的元素，table 虽然使用了 volatile 修饰，但是 volatile 并不能提供数组内部操作的可见性保证，所以源码中使用了 UNSAFE 来操作数组。
2. 如果要删除的节点不是头结点，它会将要删除节点的后继节点接到前驱节点中，这里的并发保证就是 next 属性是 volatile 的。


# 参考文献
[Map 综述（三）：彻头彻尾理解 ConcurrentHashMap](https://blog.csdn.net/justloveyou_/article/details/72783008)     
[Java7/8 中的 HashMap 和 ConcurrentHashMap 全解析](https://javadoop.com/post/hashmap)    
