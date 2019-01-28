* [处理流程](#处理流程)
    * [Map过程](#map过程)
    * [shuffle过程](#shuffle过程)
    * [Reduce过程](#reduce过程)
* [总结](#总结)
    * [分片大小确定](#分片大小确定)
    * [Map数量的确定](#map数量的确定)
    * [Shuffle后如何分配Reduce](#shuffle后如何分配reduce)
    * [Reduce数量的确定](#reduce数量的确定)
* [参考文献](#参考文献)


MapReduce是一种**适合处理大量数据的编程模型**。Hadoop能够运行用各种语言编写的MapReduce程序：Java，Ruby，Python和C++。**MapReduce程序本质上是并行的**，因此对于使用群集中的多台机器执行大规模数据分析非常有用。   

# 处理流程
MapReduce 处理数据过程主要分成 **Map** 和 **Reduce** 两个阶段。首先执行 Map 阶段，再执行 Reduce 阶段。Map 和 Reduce 的处理逻辑由用户自定义实现，但要符合 MapReduce 框架的约定。处理流程如下所示：
1. 在正式执行 Map 前，需要将输入数据进行 **分片**。所谓分片，就是将输入数据切分为大小相等的数据块，每一块作为单个 Map Task 的输入被处理，以便于多个 Map Task 同时工作。
2. 分片完毕后，多个 Map Task 便可同时工作。每个 Map Task 在读入各自的数据后，进行计算处理，最终输出给 Reduce。Map Task 在输出数据时，需要为每一条输出数据指定一个 Key，这个 Key 值决定了这条数据将会被发送给哪一个 Reduce Task。**Key 值和 Reduce Task 是多对一的关系**，具有相同 Key 的数据会被发送给同一个 Reduce Task，单个 Reduce Task 有可能会接收到多个 Key 值的数据。
3. 在进入 Reduce 阶段之前，MapReduce 框架会对数据按照 Key 值**排序**，使得具有相同 Key 的数据彼此相邻。如果您指定了 **合并操作（Combiner）**，框架会调用 Combiner，它负责对中间过程的输出具有相同 Key 的数据进行本地的聚集，这会有助于降低从Mapper到 Reducer数据传输量。Combiner 的逻辑可以由您自定义实现。这部分的处理通常也叫做 **洗牌（Shuffle）**。
4. 接下来进入 Reduce 阶段。相同 Key 的数据会到达同一个 Reduce Task。同一个 Reduce Task 会接收来自多个 Map Task 的数据。每个 Reduce Task 会对 Key 相同的多个数据进行 Reduce 操作。最后，一个 Key 的多条数据经过 Reduce 的作用后，将变成一个值。
 
下面以WordCount为例，介绍MapReduce的处理过程。   

![wordcount mapreduce处理过程](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/openmr.jpg)   

操作步骤：
1. **输入数据**：对文本进行分片，将每片内的数据作为单个 Map Task 的输入。
2. **Map 阶段**：Map 处理输入，每获取一个数字，将数字的 Count 设置为 1，并将此<Word, Count>对输出，此时以 Word 作为输出数据的 Key。
3. **Shuffle > 合并排序**：在 Shuffle 阶段前期，首先对每个 Map Task 的输出，按照 Key 值（即 Word 值）进行排序。排序后进行 Combiner 操作，即将 Key 值（Word 值）相同的 Count 累加，构成一个新的<Word, Count>对。此过程被称为合并排序。
4. **Shuffle > 分配 Reduce**：在 Shuffle 阶段后期，数据被发送到 Reduce 端。Reduce Worker 收到数据后依赖 Key 值再次对数据排序。
5. **Reduce 阶段**：每个 Reduce Task 对数据进行处理时，采用与 Combiner 相同的逻辑，将 Key 值（Word 值）相同的 Count 累加，得到输出结果。
6. **输出结果数据**。

## Map过程
- 某个TaskTracker领取map任务；
- 这个TaskTracker将作业的jar包文件和相关配置文件复制到本地工作目录下（传输jar包而不传输数据的原则，代码靠近数据的原则）；
- TaskTracker启动单独的JVM运行这个map任务；
- map的输出结果中相同的key不一定在一块，为了下面的**combine**过程简洁，这里对map输出结果中的key进行**快速排序**（**sort**过程）；
- map任务的结果存入**内存**，并在内存有限的情况下**定期写入磁盘**（**spill**过程），将某一个map的输出的相同的key的value合并（即combine过程），spill与combine经常交替进行；
- 多次spill会产生许多小的溢出文件在磁盘中，**merge**过程将他们合并；
- TaskTracker和JobTracker定期通信，报告进度。

![combiner](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/hadoop_002.png)    

![merge](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/hadoop_merge_1.png)   

## shuffle过程
- map的输出是（key, value）对（下面简称KV对）；
- 将某一个map的输出的相同的key的value合并，即**combine**过程；
- **决定这个map的结果给哪一个reduce**：通过**hash(key)mod(reduce数目)** 计算出partition的ID，上述的key就被分配给这个partition；**一个partition中可以有多个key，但是同一个key只存在于一个partition中；一个partition对应一个reduce**；
- 有的partition负载可能很重，有的则很轻，需要通过一定的协调机制平衡负载（比如根据自己的需求重写partition函数）；
- **partiton作为reduce的输入**，每个reducer获得的是每个Key在每个mapper上输出的结果，它需要使用reduce函数把相同Key的不同mapper的输出统计在一起。

## Reduce过程
- 在部分map任务执行完后（不用等到所有map任务结束）JobTracker开始分配reduce任务到TaskTracker；
- TaskTracker启动单独的JVM运行这个reduce任务；
- TaskTracker从远地下载中间结果文件到本地（指partition文件、一个partition对应一个reduce），为reduce任务真正开展做准备，但不会开始执行reduce()函数；
待所有的map任务都完成以后，JobTracker通知所有的TaskTracker开始做reduce任务；
- TaskTracker和JobTracker定期通信，报告进度；

# 总结
## 分片大小确定
通常一个split就是一个block，这样做的好处是使得Map任务可以在存储有当前数据的节点上运行本地的任务，而不需要通过网络进行跨节点的任务调度。   

可以通过设置`mapred.min.split.size`， `mapred.max.split.size`, `block.size`来控制拆分的大小。如果`mapred.min.split.size`大于block size，则会将两个block合成到一个split，这样有部分block数据需要通过网络读取；如果`mapred.max.split.size`小于block size，则会将一个block拆成多个split，增加了Map任务数。   

假设splitSize是默认的64M，现在输入包含3个文件，这3个文件的大小分别为10M，64M，100M，那么这3个文件会被分割为：

```
输入文件大小                10M     64M     100M
分割后的InputSplit大小      10M     64M     64M，36M
```

在Map任务开始前，会先获取文件在HDFS上的路径和block信息，然后根据splitSize对文件进行切分（`splitSize = computeSplitSize(blockSize, minSize, maxSize)` ），默认splitSize 就等于blockSize的默认值（64m）。

## Map数量的确定
用户通过指定期望的map数和期望的分片最小值来调控map数量，具体是系统通过下面几步算出map个数：   
![map个数](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/map.png)   
map正常的并行规模大致是每个节点（node）大约10到100个map，对于CPU 消耗较小的map任务可以设到300个左右。Map任务的个数也能通过使用`JobConf 的conf.setNumMapTasks(int num)`方法来手动地设置。   

合适Map数量的好处：
- **减少了调度的负担**；更少的map意味着任务调度更简单，集群中可用的空闲槽更多。
- 有足够的内存将map输出容纳在排序缓存中，这**使map端更有效率**；
- **减少了需要shuffle map输出的寻址次数**，每个map产生的输出可用于每一个reduce，因此寻址数就是map个数乘以reduce个数；
- 这使**reduce端合并map输出的过程更高效**，因为需要合并的文件段更少了，所以合并的次数更少。

## Shuffle后如何分配Reduce
通过使用**Partitioner**划分键值空间（key space）。   
Partitioner负责控制map输出结果key的分割。Key（或者一个key子集）被用于产生分区，通常使用的是Hash函数。分区的数目与一个作业的reduce任务的数目是一样的。因此，它控制将中间过程的key（也就是这条记录）应该发送给m个reduce任务中的哪一个来进行reduce操作。   
**HashPartitioner是默认的 Partitioner**。   

## Reduce数量的确定
Reducer的个数是由用户独立设置的，在默认情况下只有一个Reducer。 它的个数既可以使用命令行参数设置（`mapreduce.job.reduces=number`），也可以在程序中制定（`job.setNumReduceTasks(number)`）。   

Reducer个数应该设置为**0.95或者1.75乘以节点数与每个节点的容器数的乘积**。 
- 当乘数为0.95时，map任务结束后所有的reduce将会立刻启动并开始转移数据， 此时队列中无等待任务，该设置适合reudce任务执行时间短或者reduce任务在个节点的执行时间相差不大的情况； 
- 当乘数为1.75时，运行较快的节点将在完成第一轮reduce任务后，可以立即从队列中取出新的reduce任务执行， 

由于该reduce个数设置方法减轻了单个reduce任务的负载，并且运行较快的节点将执行新的reduce任务而非空等执行较慢的节点，其拥有更好的负载均衡特性。
reduces的性能很大程度上受shuffle的性能所影响。**太少的reduce会使得reduce运行的节点处于过度负载状态，太多的reduce对shuffle过程有不利影响，会导致作业的输出都是些小文件。**

# 参考文献
[Hadoop Map/Reduce教程——官方文档](http://hadoop.apache.org/docs/r1.0.4/cn/mapred_tutorial.html)   
[Hadoop Map/Reduce执行流程详解](http://zheming.wang/blog/2015/05/19/3AFF5BE8-593C-4F76-A72A-6A40FB140D4D/)   
[MapReduce过程详解](https://changsiyuan.github.io/2015/04/01/2015-4-1-mapreduce/)   
[What is MapReduce? How it Works - Hadoop MapReduce Tutorial](https://www.guru99.com/introduction-to-mapreduce.html)   
[MapReduce——阿里云](https://www.alibabacloud.com/help/zh/doc-detail/27875.htm)   
[Hadoop中的Mapper和Reducer数量设定](http://summerisgreen.com/blog/2018-04-24-2018-04-23-hadoop%E4%B8%ADmapper%E5%92%8Creducer%E4%B8%AA%E6%95%B0%E7%9A%84%E8%AE%BE%E7%BD%AE.html)
