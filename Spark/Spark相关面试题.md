[toc]

# 介绍下Spark任务提交之后发生了什么
1. 用户通过spark-submit脚本提交应用。
2. spark-submit根据用户代码及配置**确定使用哪个资源管理器**，以及**在合适的位置启动driver**。
3. driver与集群管理器(如YARN)通信，申请资源以启动executor。
4. 集群管理器启动executor。
5. driver进程执行用户的代码，根据程序中定义的transformation和action，进行stage的划分，然后以task的形式发送到executor。（通过DAGScheduler划分stage，通过TaskScheduler和TaskSchedulerBackend来真正申请资源运行task）
6. task在executor中进行计算并保存结果。
7. 如果driver中的main()方法执行完成则退出，或者调用了SparkContext#stop()，driver会终止executor进程，并且通过集群管理器释放资源。

![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/spark%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png)

# Spark的运行模式以及区别
1. 本地模式：Spark单机运行，一般用于开发测试。
2. Standalone模式：构建一个由Master+Slave构成的Spark集群，Spark运行在集群中。
3. Spark on Yarn模式：Spark客户端直接连接Yarn。不需要额外构建Spark集群。
4. Spark on Mesos模式：Spark客户端直接连接Mesos。不需要额外构建Spark集群。

spark-submit脚本在Spark的bin目录下，可以利用此脚本向集群提交Spark应用。

```
./bin/spark-submit \
  --class <main-class>
  --master <master-url> \
  --deploy-mode <deploy-mode> \
  --conf <key>=<value> \
  ... # 其他选项
  <application-jar> \
  [application-arguments]
```

一些常用的选项如下：
- --class: 应用入口类（例如：org.apache.spark.examples.SparkPi)）
- --master: 集群的master URL （如：spark://23.195.26.187:7077）
- --deploy-mode: 驱动器进程是在集群上工作节点运行（cluster），还是在集群之外客户端运行（client）（默认：client）
- --conf: 可以设置任意的Spark配置属性，键值对（key=value）格式。如果值中包含空白字符，可以用双引号括起来（”key=value“）。
- application-jar: 应用程序jar包路径，该jar包必须包括你自己的代码及其所有的依赖项。如果是URL，那么该路径URL必须是对整个集群可见且一致的，如：hdfs://path 或者 file://path （要求对所有节点都一致）
- application-arguments: 传给入口类main函数的启动参数，如果有的话。

实例：    

```
# 本地运行，占用8个core
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master local[8] \
  /path/to/examples.jar \
  100

# 独立部署，client模式
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://207.184.161.138:7077 \
  --executor-memory 20G \
  --total-executor-cores 100 \
  /path/to/examples.jar \
  1000

# 独立部署，cluster模式，异常退出时自动重启
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://207.184.161.138:7077 \
  --deploy-mode cluster
  --supervise
  --executor-memory 20G \
  --total-executor-cores 100 \
  /path/to/examples.jar \
  1000

# YARN上运行，cluster模式
export HADOOP_CONF_DIR=XXX
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master yarn \
  --deploy-mode cluster \  # 要client模式就把这个设为client
  --executor-memory 20G \
  --num-executors 50 \
  /path/to/examples.jar \
  1000

# Mesos集群上运行，cluster模式，异常时自动重启
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master mesos://207.184.161.138:7077 \
  --deploy-mode cluster
  --supervise
  --executor-memory 20G \
  --total-executor-cores 100 \
  http://path/to/examples.jar \
  1000
```

传给Spark的master URL可以是以下几种格式：    
![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/masterurl.png)

## 参考文献
[Spark的四种运行模式(1.2.1)](http://blog.cheyo.net/29.html)    
[《Spark官方文档》提交Spark应用](http://ifeve.com/spark-submit/)

# Spark任务提交时的常用参数配置
- **num-executors**：该参数用于设置Spark作业总共要用多少个Executor进程来执行。每个Spark作业的运行一般设置**50~100**个左右的Executor进程比较合适。
- **executor-memory**：该参数用于设置每个Executor进程的内存。每个Executor进程的内存设置**4G\~8G**较为合适。**num-executors乘以executor-memory，是不能超过队列的最大内存量的**。**如果你是跟团队里其他人共享这个资源队列**，那么申请的内存量最好不要超过资源队列最大总内存的**1/3\~1/2**，避免你自己的Spark作业占用了队列所有的资源，导致别的同学的作业无法运行。
- **executor-cores**：该参数用于设置每个Executor进程的CPU core数量。Executor的CPU core数量设置为**2~4**个较为合适。同样得根据不同部门的资源队列来定，可以看看自己的资源队列的最大CPU core限制是多少，再依据设置的Executor数量，来决定每个Executor进程可以分配到几个CPU core。**如果是跟他人共享这个队列**，那么`num-executors * executor-cores`不要超过队列总CPU core的**1/3~1/2**左右比较合适，也是避免影响其他同学的作业运行。
- **driver-memory**：该参数用于设置Driver进程的内存。Driver的内存通常来说不设置，或者设置**1G左右**应该就够了。唯一需要注意的一点是，如果需要使用collect算子将RDD的数据全部拉取到Driver上进行处理，那么必须确保Driver的内存足够大，否则会出现OOM内存溢出的问题。
- **spark.default.parallelism**：该参数用于设置每个stage的默认task数量。Spark作业的默认task数量为**500~1000**个较为合适。如果不去设置这个参数，那么此时就会导致Spark自己根据底层HDFS的block数量来设置task的数量，默认是一个HDFS block对应一个task。设置该参数为`num-executors * executor-cores`的**2~3倍**较为合适，比如Executor的总CPU core数量为300个，那么设置1000个task是可以的。
- **spark.storage.memoryFraction**：该参数用于设置RDD持久化数据在Executor内存中能占的比例，**默认是0.6**。也就是说，默认Executor 60%的内存，可以用来保存持久化的RDD数据。
    - 如果Spark作业中，有较多的RDD持久化操作，该参数的值可以适当提高一些，保证持久化的数据能够容纳在内存中。
    - 如果Spark作业中的shuffle类操作比较多，而持久化操作比较少，那么这个参数的值适当降低一些比较合适。
    - 如果发现作业由于频繁的gc导致运行缓慢（通过spark web ui可以观察到作业的gc耗时），意味着task执行用户代码的内存不够用，那么同样建议调低这个参数的值。
- **spark.shuffle.memoryFraction**：该参数用于设置shuffle过程中一个task拉取到上个stage的task的输出后，进行聚合操作时能够使用的Executor内存的比例，**默认是0.2**。也就是说，Executor默认只有20%的内存用来进行该操作。

以下是一份spark-submit命令的示例，大家可以参考一下，并根据自己的实际情况进行调节：
```
./bin/spark-submit \
  --master yarn-cluster \
  --num-executors 100 \
  --executor-memory 6G \
  --executor-cores 4 \
  --driver-memory 1G \
  --conf spark.default.parallelism=1000 \
  --conf spark.storage.memoryFraction=0.5 \
  --conf spark.shuffle.memoryFraction=0.3 \
```
## 参考文献
[Spark性能优化指南——基础篇](https://tech.meituan.com/spark_tuning_basic.html)

# Spark算子中groupByKey和reduceByKey的区别
参见[【Spark系列2】reduceByKey和groupByKey区别与用法](https://blog.csdn.net/zongzhiyuan/article/details/49965021)、[在Spark中尽量少使用GroupByKey函数](https://www.iteblog.com/archives/1357.html)和[Spark 中 groupByKey、reduceByKey 的区别](https://ooon.me/2016/04/spark-reducebykey-groupbykey/)   
- reduceByKey(func, numPartitions=None)：reduceByKey用于对每个key对应的多个value进行merge操作，最重要的是**它能够在本地先进行merge操作**，并且merge操作可以通过函数自定义。
- groupByKey(numPartitions=None)：groupByKey也是对每个key进行操作，但只生成一个sequence。如果需要对sequence进行aggregation操作（注意，groupByKey本身不能自定义操作函数），那么，选择reduceByKey/aggregateByKey更好。这是因为groupByKey不能自定义函数，我们需要先用groupByKey生成RDD，然后才能对此RDD通过map进行自定义函数操作。    

1. 当采用reduceByKeyt时，Spark可以在每个分区移动数据之前将待输出数据与一个共用的key结合。    
![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/reduceByKey.jpg) 
2. 当采用groupByKey时，由于它不接收函数，spark只能先将所有的键值对(key-value pair)都移动，这样的后果是集群节点之间的开销很大，导致传输延时。   
![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/groupByKey.jpg)

**因此，在对大数据进行复杂计算时，reduceByKey优于groupByKey。**

# spark如何读取压缩文件（lzo）


# spark存储到hive怎么存储的

```java
spark.sql("use databasename")
// 做一些关于hive的设置
spark.sql("...")
// 将Dataframe创建成一个临时视图，供接下来的查询操作
df.createOrReplaceTempView("tmp")
// 插入hive表
spark.sql("drop table if exists tmp_countnum");
spark.sql("create table tmp_countnum as select gbId,drawRandomItems,items,item,date from temp");
```
也可以使用saveAsTable()，创建hive的Internal Table。

```scala
/** 
 * 通过HiveContext使用join直接基于hive中的两种表进行操作 
 */  
val resultDF = hiveContext.sql("select pi.name,pi.age,ps.score "  
                  +" from people pi join peopleScores ps on pi.name=ps.name"  
                  +" where ps.score>90");  

/** 
 * 通过saveAsTable创建一张hive managed table，数据的元数据和数据即将放的具体位置都是由 
 * hive数据仓库进行管理的，当删除该表的时候，数据也会一起被删除（磁盘的数据不再存在） 
 */  
hiveContext.sql("drop table if exists peopleResult")  
resultDF.saveAsTable("peopleResult") 
```

# Spark和Hadoop对比
参见[Spark、Hadoop、Storm对比](https://wlypku.github.io/2017/03/11/spark-hadoop-storm/)
- 抽象层次低，需要手工编写代码来完成，使用上难以上手。
    - =>基于RDD的抽象，实数据处理逻辑的代码非常简短。。
- 只提供两个操作，Map和Reduce，表达力欠缺。
    - =>提供很多转换和动作，很多基本操作如Join，GroupBy已经在RDD转换和动作中实现。
- 一个Job只有Map和Reduce两个阶段（Phase），复杂的计算需要大量的Job完成，Job之间的依赖关系是由开发者自己管理的。
    - =>一个Job可以包含RDD的多个转换操作，在调度时可以生成多个阶段（Stage），而且如果多个map操作的RDD的分区不变，是可以放在同一个Task中进行。
-  处理逻辑隐藏在代码细节中，没有整体逻辑
    - =>在Scala中，通过匿名函数和高阶函数，RDD的转换支持流式API，可以提供处理逻辑的整体视图。代码不包含具体操作的实现细节，逻辑更清晰。
- 中间结果也放在HDFS文件系统中
    - =>中间结果放在内存中，内存放不下了会写入本地磁盘，而不是HDFS。
- ReduceTask需要等待所有MapTask都完成后才可以开始
    - => 分区相同的转换构成流水线放在一个Task中运行，分区不同的转换需要Shuffle，被划分到不同的Stage中，需要等待前面的Stage完成后才可以开始。
- 时延高，只适用Batch数据处理，对于交互式数据处理，实时数据处理的支持不够
    - =>通过将流拆成小的batch提供Discretized Stream处理流数据。
- 对于迭代式数据处理性能比较差
    - =>通过在内存中缓存数据，提高迭代式计算的性能。

# Spark容错机制和Hadoop容错机制的区别
## Spark容错机制
RDD只支持**粗粒度转换**，即只记录单个块上执行的单个操作，然后将创建RDD的一系列变换序列（每个RDD都包含了他是如何由其他RDD变换过来的以及如何重建某一块数据的信息。RDD的容错机制又称 **“血统(Lineage)”容错**）    

RDD的Lineage记录的是粗颗粒度的特定数据Transformation操作（如filter、map、join等）行为。当这个RDD的部分分区数据丢失时，它可以通过Lineage获取足够的信息来重新运算和恢复丢失的数据分区。

RDD在Lineage依赖方面分为两种：**窄依赖**(Narrow Dependencies)与**宽依赖**(Wide Dependencies,源码中称为Shuffle 
Dependencies)，用来解决数据容错的高效性。    

1. 窄依赖可以在某个计算节点上直接通过计算父RDD的某块数据计算得到子RDD对应的某块数据；宽依赖则要等到父RDD所有数据都计算完成之后，并且父RDD的计算结果进行hash并传到对应节点上之后才能计算子RDD。 
2. 数据丢失时，对于窄依赖只需要重新计算丢失的那一块数据来恢复；对于宽依赖则要将祖先RDD中的所有数据块全部重新计算来恢复。所以在长“血统”链特别是有宽依赖的时候，需要在适当的时机设置**数据检查点**。也是这两个特性要求对于不同依赖关系要采取不同的任务调度机制和容错恢复机制。

- 在窄依赖中，在子RDD的分区丢失、重算父RDD分区时，父RDD相应分区的所有数据都是子RDD分区的数据，并不存在冗余计算。
- 在宽依赖情况下，丢失一个子RDD分区重算的每个父RDD的每个分区的所有数据并不是都给丢失的子RDD分区用的，会有一部分数据相当于对应的是未丢失的子RDD分区中需要的数据，这样就会产生冗余计算开销，这也是宽依赖开销更大的原因。


通过上述分析可以看出在以下两种情况下，RDD需要加检查点。
1. DAG中的**Lineage过长**，如果重算，则开销太大（如在PageRank中）。
2. 在**宽依赖**上做Checkpoint获得的收益更大。 

**检查点**（本质是通过将RDD写入Disk做检查点）是为了通过lineage做容错的辅助，lineage过长会造成容错成本过高，这样就不如在中间阶段做检查点容错，如果之后有节点出现问题而丢失分区，从做检查点的RDD开始重做Lineage，就会减少开销。

## Hadoop容错机制
### MRv1
#### Task任务出错
tasktracker会将**释放此任务占用的槽以便运行另外一个任务**。jobtracker接到任务失败的通知后，**会将其重新加入到调度队列重新分配给其他的tasktracker执行**，默认情况每个失败任务尝试**4次**后仍然没有完成，就不会再重试（jobtracker会将其标记为killed)，此时整个作业就执行失败了。

#### TaskTracker失败
- tasktracker上运行的任务将会被移送到其他tasktracker节点上去运行。
- 如果成功完成的map任务，tasktracker节点已经失效了，那么reduce任务也无法访问到存储在tasktracker本地文件系统上的中间结果，需要在其他tasktracker节点重新被执行。
- 如果是reduce阶段呢，如果是reduce阶段自然就是执行为完成的reduce任务了，因为reduce只要执行完了的就会把输出写到Hdfs上。

#### jobtracker失败
jobtracker失败应该说是最严重的一种失败方式了，而且在hadoop中存在单点故障的情况下是相当严重的，因为在这种情况下作业最终失败。

### MRv2
#### Task Failure
task 失败是在map或reduce task中的用户代码抛出一个运行时异常。**application master将task标记为失败，并且释放container**，以便其资源可用于其它的task。application mastet会尽量避免在之前失败的同一个节点上重新调度task。此外，如果一个task失败4次，将不会被再次重试。这个次数是可以配置的。

#### Application Master Failure
在YARN中的应用如果失败了也会重新尝试运行。尝试运行MapReduce application master的最大次数是由mapreduce.am.max-attempts属性控制的，默认值为2。

#### Node Manager Failure
如果一个node manage节点因中断或运行缓慢而失败，那么它将不会发送心跳到resource manager。如果resource manage在10分钟内没有接收到一个心跳，它会感知到node manager已经停了，并把它从节点集群中移除。

在出现故障的node manager节点上运行的任何task或application master将会按上两节描述的机制来恢复。另外，对于出现故障的node manager节点，application master分配在其上已经成功运行完成的map task，**如果这些map task属于未完成的Job，它们将会被重新运行**，因为它们的中间输出存放在故障node manager节点的本地文件系统中，reduce task可能不能访问。

**黑名单机制**：黑名单是由application master管理的。

#### Resource Manager Failure
Resource manager出现故障是比较严重的，因为没有它，job 和 task都不能被启动。默认配置，resource manager是一个单点故障，因为在机器出现故障时，所有的job都会失败，并且不能被恢复。

## 参考文献
[RDD之七：Spark容错机制](https://www.cnblogs.com/duanxz/p/6329675.html)    
[【Spark】Spark容错机制](https://blog.csdn.net/JasonDing1354/article/details/46882585)   
[MapReduce错误任务失败处理](https://blog.csdn.net/luyee2010/article/details/8715135)   
[MapReduce的容错机制](https://blog.csdn.net/qiruiduni/article/details/48285593)   
[MapReduce错误处理，任务调度及Shuffle过程](https://blog.csdn.net/scgaliguodong123_/article/details/46439219)



# 为什么Spark比MapReduce快
参见[为什么Spark比MapReduce快？](https://www.zhihu.com/question/31930662/answer/155447324)

这俩根本没啥可比的，能够单MR做完的任务，Spark未必比MR快。至于迭代不迭代的并不是关键，其实你在Mapper里对数据做N个操作基本等价于N个窄依赖RDD的连接。    

**所以说真要比，也是多个MR组成的复杂Job来和Spark比。**

MR由于其计算粒度的设计问题，在进行需要多次MR组合的计算时，每次MR除了Shuffle的磁盘开销外，Reduce之后也会写到磁盘。

而Spark的DAG实质上就是把计算和计算之间的编排变得更为细致紧密，**使得很多MR任务中需要落盘的非Shuffle操作得以在内存中直接参与后续的运算**，并且由于算子粒度和算子之间的逻辑关系使得其易于由框架自动地优化（换言之编排得好的MR其实也可以做到）。

另外在进行复杂计算任务的时候，Spark的错误恢复机制在很多场景会比MR的错误恢复机制的代价低，这也是性能提升的一个点。

参见[Spark为什么比Hadoop快？](https://blog.csdn.net/Stefan_xiepj/article/details/80347720)

Spark和Hadoop的根本差异是多个任务之间的数据通信问题：**Spark多个stage之间数据通信是基于内存，而Hadoop复杂的Job需要多个Job共同实现，其中数据通信是基于磁盘。**
Spark只有在shuffle的时候将数据写入磁盘，而Hadoop中多个MR作业之间的数据交互都要依赖于磁盘交互。

Spark为**每个应用程序在worker上开启一个进程，而一个Job中的Task会在同一个线程池中运行**，而Hadoop Map Reduce的计算模型**是每个Task(Mapper或者Reducer）都是一个单独的进程，启动停止进程非常expensive，同时，进程间的数据共享也不能基于内存，只能是HDFS**。

# spark中cache和checkpoint的区别，cache存储在内存中会不会抛出内存溢出异常
不会抛出异常，因为cache()的存储级别是“memory_only”，该级别的说明如下：
> 将 RDD 以反序列化的 Java 对象的形式存储在 JVM 中. **如果内存空间不够，部分数据分区将不再缓存**，在每次需要用到这些数据时重新进行计算. 这是默认的级别.

# spark统一内存管理中spark.storage.memoryFraction设置为0.5代表什么？
## 堆内
> 可用的存储内存 = Unified Memory(统一内存) * spark.storage.memoryFraction   
Unified Memory(统一内存) = spark.memory.fraction(spark 2.0+ 中默认是0.6) * 可用内存    
可用内存 = 系统内存 - 预留内存(默认是300M)

## 堆外
> storage内存 = spark.storage.memoryFraction * 堆外可用内存

所以最后的**storage内存 = 堆内storage内存 + 堆外storage内存**

默认情况下，**堆外内存是关闭的**，我们可以通过 `spark.memory.offHeap.enabled` 参数启用，并且通过 `spark.memory.offHeap.size` 设置堆外内存大小，**单位为字节**。

# spark在进行Shuffle操作时reduce task的个数如何确定？hashpartitioner()传入参数会影响reduce的个数吗？
reduce task的个数由stage的第一个RDD(shuffledRDD)的partition数量决定，而shuffledRDD的partition数取决于Partitioner(分区器)中的partition的个数。

HashPartitioner中partition的数量：
- **spark.default.parallelism**：用于设置作业的并行度。该参数用于设置每个stage的默认task数量。
- 如果没有设置上面的参数，则由父RDD中分区数最大的决定。

参见[Spark分区器HashPartitioner和RangePartitioner代码详解](https://www.iteblog.com/archives/1522.html)和[spark中的mapper和reducer个数是否可以配置？](https://www.zhihu.com/question/40882893)    