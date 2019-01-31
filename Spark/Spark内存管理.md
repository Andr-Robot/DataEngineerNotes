* [静态内存管理](#静态内存管理)
    * [堆内内存](#堆内内存)
    * [堆外内存](#堆外内存)
    * [总结](#总结)
* [统一内存管理](#统一内存管理)
* [参考文献](#参考文献)

# 静态内存管理
## 堆内内存
在 Spark 最初采用的**静态内存管理机制**，存储内存、执行内存和其他内存的大小在 Spark 应用程序运行期间均为固定的，但用户可以应用程序启动前进行配置，堆内内存的分配如下所示：    

![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/staticmemorymanagerinheap.png)    

可以看到，可用的堆内内存的大小需要按照下面的方式计算：
> **可用的存储内存 = systemMaxMemory * spark.storage.memoryFraction * spark.storage.safetyFraction**     
**可用的执行内存 = systemMaxMemory * spark.shuffle.memoryFraction * spark.shuffle.safetyFraction**

其中 **systemMaxMemory** 取决于**当前 JVM 堆内内存的大小**，最后可用的执行内存或者存储内存要在此基础上与各自的 memoryFraction 参数和 safetyFraction 参数相乘得出。上述计算公式中的两个 safetyFraction 参数，其意义在于在逻辑上预留出 safetyFraction 这么一块保险区域，降低因实际内存超出当前预设范围而导致 OOM 的风险。

**注意**：这个预留的保险区域仅仅是一种逻辑上的规划，在具体使用时 Spark 并没有区别对待，和"其它内存"一样交给了 JVM 去管理。

## 堆外内存
堆外的空间分配较为简单，只有**存储内存和执行内存**，如下图所示。可用的执行内存和存储内存占用的空间大小直接由参数 spark.memory.storageFraction 决定，由于堆外内存占用的空间可以被精确计算，所以无需再设定保险区域。

![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/staticmemorymanagerinnoheap.png)

## 总结
- 优点：静态内存管理机制实现起来较为简单
- 缺点：会出现存储内存和执行内存中的一方剩余大量的空间，而另一方却早早被占满，不得不淘汰或移出旧的内容以存储新的内容。

# 统一内存管理
Spark 1.6 之后引入的统一内存管理机制，与静态内存管理的区别在于存储内存和执行内存共享同一块空间，可以动态占用对方的空闲区域，如下图所示：

![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/unifiedmemorymanagerinheap.png)

> **systemMemory = spark.executor.memory**    
**reservedMemory = 300MB**      
**usableMemory = systemMemory - reservedMemory**     
**StorageMemory= usableMemory * spark.memory.fraction * spark.memory.storageFraction**    

![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/unifiedmemorymanagerinnoheap.png)

其中最重要的优化在于动态占用机制，其规则如下：
- 设定基本的存储内存和执行内存区域（spark.storage.storageFraction 参数），该设定确定了双方各自拥有的空间的范围
- 双方的空间都不足时，则存储到硬盘；若己方空间不足而对方空余时，可借用对方的空间;（存储空间不足是指不足以放下一个完整的 Block）
- **执行内存的空间被对方占用后，可让对方将占用的部分转存到硬盘，然后"归还"借用的空间**
- **存储内存的空间被对方占用后，无法让对方"归还"**，因为需要考虑 Shuffle 过程中的很多因素，实现起来较为复杂

![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/unifiedmemorymanager.png)

# 参考文献
[Apache Spark 内存管理详解](https://www.ibm.com/developerworks/cn/analytics/library/ba-cn-apache-spark-memory-management/index.html)     
[Apache Spark 统一内存管理模型详解](https://www.iteblog.com/archives/2342.html)     
