[toc]

# HashMap的定义

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    // 默认的容量大小，该大小必须是2的幂
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    // 最大容量，2的30次方（若传入的容量过大，将被最大值替换）
    static final int MAXIMUM_CAPACITY = 1 << 30;

    // 加载因子：HashMap在其容量自动增加前可达到多满的一种尺度
    // a. 加载因子越大、填满的元素越多 = 空间利用率高、但冲突的机会加大、查找效率变低（因为链表变长了）
    // b. 加载因子越小、填满的元素越少 = 空间利用率小、冲突的机会减小、查找效率高（链表不长）
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    // 一个桶的树化阈值
    // 当桶中元素个数超过这个值时，需要使用红黑树节点替换链表节点
    // 这个值必须为 8，要不然频繁转换效率也不高
    static final int TREEIFY_THRESHOLD = 8;

    // 一个树的链表还原阈值
    // 当扩容时，桶中元素个数小于这个值，就会把树形的桶元素 还原（切分）为链表结构
    // 这个值应该比上面那个小，至少为 6，避免频繁转换
    static final int UNTREEIFY_THRESHOLD = 6;

    // 哈希表的最小树形化容量
    // 当哈希表中的容量大于这个值时，表中的桶才能进行树形化
    // 否则桶内元素太多时会扩容，而不是树形化
    // 为了避免进行扩容、树形化选择的冲突，这个值不能小于 4 * TREEIFY_THRESHOLD
    static final int MIN_TREEIFY_CAPACITY = 64;
}
```

HashMap继承AbstractMap类，实现了Map、Cloneable和Serializable接口。

# HashMap的构造方法
## HashMap()
该构造函数是默认构造函数，意在构造一个具有：**默认初始容量 (16) 和 默认负载因子(0.75)** 的空 HashMap

```java
/**
 * Constructs an empty <tt>HashMap</tt> with the default initial capacity
 * (16) and the default load factor (0.75).
 */
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```


## HashMap(int initialCapacity)
该构造函数意在构造**一个指定初始容量和默认负载因子 (0.75)** 的空 HashMap

```java
/**
 * Constructs an empty <tt>HashMap</tt> with the specified initial
 * capacity and the default load factor (0.75).
 *
 * @param  initialCapacity the initial capacity.
 * @throws IllegalArgumentException if the initial capacity is negative.
 */
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

## HashMap(int initialCapacity, float loadFactor)
指定“容量大小”和“加载因子”的构造函数。

```
/**
 * Constructs an empty <tt>HashMap</tt> with the specified initial
 * capacity and load factor.
 *
 * @param  initialCapacity the initial capacity
 * @param  loadFactor      the load factor
 * @throws IllegalArgumentException if the initial capacity is negative
 *         or the load factor is nonpositive
 */
public HashMap(int initialCapacity, float loadFactor) {
    // 初始容量不能小于 0
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    // 与最大的容量对比，取两者中最小
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    // 负载因子不能小于 0，也不能为空
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    // threshold是HashMap判断size是否需要扩容的阈值
    // tableSizeFor方法保证函数返回值是大于等于给定参数initialCapacity最小的2的幂次方的数值。
    this.threshold = tableSizeFor(initialCapacity);
}
```


## HashMap(Map<? extends K, ? extends V> m)
包含“子Map”的构造函数。

```java 
/**
 * Constructs a new <tt>HashMap</tt> with the same mappings as the
 * specified <tt>Map</tt>.  The <tt>HashMap</tt> is created with
 * default load factor (0.75) and an initial capacity sufficient to
 * hold the mappings in the specified <tt>Map</tt>.
 *
 * @param   m the map whose mappings are to be placed in this map
 * @throws  NullPointerException if the specified map is null
 */
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

# HashCode()

```java
public final int hashCode() {
    return Objects.hashCode(key) ^ Objects.hashCode(value);
}
```

# HashMap保存数据的过程
1. 首先，判断key是否为null；
    - 若为null，则直接调用putForNullKey方法；
    - 若不为空，则先计算key的hash值；
2. 然后根据hash值搜索在table数组中的索引位置;
3. 如果table数组在该位置处有元素，则查找是否存在相同的key，若存在则覆盖原来key的value；
4. 否则将该元素保存在链头（最先保存的元素放在链尾）。此外，若table在该处没有元素，则直接保存。

# HashMap读取数据的过程
HashMap只需通过key的hash值定位到table数组的某个特定的桶，然后查找并返回该key对应的value即可。

调用HashMap的get(Object key)方法后，若返回值是 NULL，则存在如下两种可能：
- 该 key 对应的值就是 null;
- HashMap 中不存在该 key。


# HashMap 的底层数组长度为何总是2的n次方？
- 不同的hash值发生碰撞的概率比较小，这样就会使得数据在table数组中分布较均匀，空间利用率较高，查询速度也较快；
- h&(length - 1) 就相当于对length取模，而且在速度、效率上比直接取模要快得多，即二者是等价不等效的，这是HashMap在速度和效率上的一个优化。






# 参考文献
[源码分析之 HashMap](https://juejin.im/post/58f2f47061ff4b0058f4b7cc)    
[Java HashMap工作原理及实现](https://yikun.github.io/2015/04/01/Java-HashMap%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)   
[Java：手把手带你源码分析 HashMap 1.7](https://blog.csdn.net/carson_ho/article/details/79373026)   
[Map 综述（一）：彻头彻尾理解 HashMap](https://blog.csdn.net/justloveyou_/article/details/62893086)      
[Java 集合系列10之 HashMap详细介绍(源码解析)和使用示例](http://www.cnblogs.com/skywang12345/p/3310835.html)    
[Java 集合深入理解（16）：HashMap 主要特点和关键方法源码解读](https://blog.csdn.net/u011240877/article/details/53351188)
