* [数据倾斜](#数据倾斜)
* [如何定位导致数据倾斜的代码](#如何定位导致数据倾斜的代码)
* [查看导致数据倾斜的key的数据分布情况](#查看导致数据倾斜的key的数据分布情况)
* [数据倾斜的解决方案](#数据倾斜的解决方案)
    * [1. 使用Hive ETL预处理数据](#1-使用hive-etl预处理数据)
    * [2. 过滤少数导致倾斜的key](#2-过滤少数导致倾斜的key)
    * [3. 自定义Partitioner](#3-自定义partitioner)
    * [4. 提高shuffle操作的并行度](#4-提高shuffle操作的并行度)
    * [5. 两阶段聚合（局部聚合+全局聚合）](#5-两阶段聚合局部聚合全局聚合)
    * [6. 将reduce join转为map join](#6-将reduce-join转为map-join)
    * [7. 采样倾斜key并分拆join操作](#7-采样倾斜key并分拆join操作)
    * [8. 使用随机前缀和扩容RDD进行join](#8-使用随机前缀和扩容rdd进行join)
    * [9. 多种方案组合使用](#9-多种方案组合使用)
* [参考文献](#参考文献)

# 数据倾斜
数据倾斜指的是，并行处理的数据集中，某一部分（如Spark或Kafka的一个Partition）的数据显著多于其它部分，从而使得该部分的处理速度成为整个数据集处理的瓶颈。

# 如何定位导致数据倾斜的代码
**数据倾斜只会发生在shuffle过程中**。这里给大家罗列一些常用的并且可能会触发shuffle操作的算子：**distinct、groupByKey、reduceByKey、aggregateByKey、join、cogroup、repartition等**。出现数据倾斜时，可能就是你的代码中使用了这些算子中的某一个所导致的。
1. 查看Spark Web UI中每个stage的各个task的统计值（运行时间或处理数据量），从而进一步确定是不是task分配的数据不均匀导致了数据倾斜。
2. 知道数据倾斜发生在哪一个stage之后，接着我们就需要根据stage划分原理，推算出来发生倾斜的那个stage对应代码中的哪一部分，这部分代码中肯定会有一个shuffle类算子。

# 查看导致数据倾斜的key的数据分布情况
知道了数据倾斜发生在哪里之后，通常需要分析一下那个执行了shuffle操作并且导致了数据倾斜的RDD/Hive表，查看一下其中key的分布情况。   

此时根据你执行操作的情况不同，可以有很多种查看key分布的方式：   
1. 如果是Spark SQL中的group by、join语句导致的数据倾斜，那么就查询一下SQL中使用的表的key分布情况。  
2. 如果是对Spark RDD执行shuffle算子导致的数据倾斜，那么可以在Spark作业中加入查看key分布的代码，比如RDD.countByKey()。然后对统计出来的各个key出现的次数，collect/take到客户端打印一下，就可以看到key的分布情况。

    ```scala
    # 假如统计pairs这个RDD中key的分布情况，可以先采样10%的样本数据
    val sampledPairs = pairs.sample(false, 0.1)
    # 使用countByKey算子统计出每个key出现的次数
    val sampledWordCounts = sampledPairs.countByKey()
    # 在客户端遍历和打印样本数据中各个key的出现次数
    sampledWordCounts.foreach(println(_))
    ```

# 数据倾斜的解决方案
## 1. 使用Hive ETL预处理数据
**方案适用场景：** 导致数据倾斜的是Hive表。如果该Hive表中的数据本身很不均匀（比如某个key对应了100万数据，其他key才对应了10条数据），而且业务场景需要频繁使用Spark对Hive表执行某个分析操作，那么比较适合使用这种技术方案。
## 2. 过滤少数导致倾斜的key
**方案适用场景：** 如果发现导致倾斜的key就少数几个，而且对计算本身的影响并不大的话，那么很适合使用这种方案。比如99%的key就对应10条数据，但是只有一个key对应了100万数据，从而导致了数据倾斜。
## 3. 自定义Partitioner
使用自定义的Partitioner（默认为HashPartitioner），将原本被分配到同一个Task的不同Key分配到不同Task。
## 4. 提高shuffle操作的并行度
**方案适用场景：** 如果我们必须要对数据倾斜迎难而上，那么建议优先使用这种方案，因为这是处理数据倾斜最简单的一种方案。   

增加shuffle read task的数量，可以让原本分配给一个task的多个key分配给多个task，从而让每个task处理比原来更少的数据。
## 5. 两阶段聚合（局部聚合+全局聚合）
**方案适用场景：** 对RDD执行reduceByKey等聚合类shuffle算子或者在Spark SQL中使用group by语句进行分组聚合时，比较适用这种方案。   

将原本相同的key通过附加随机前缀的方式，变成多个不同的key，就可以让原本被一个task处理的数据分散到多个task上去做局部聚合，进而解决单个task处理数据量过多的问题。接着去除掉随机前缀，再次进行全局聚合，就可以得到最终的结果。
## 6. 将reduce join转为map join
**方案适用场景：** 在对RDD使用join类操作，或者是在Spark SQL中使用join语句时，而且join操作中的一个RDD或表的数据量比较小（比如几百M或者一两G），比较适用此方案。   

普通的join是会走shuffle过程的，而一旦shuffle，就相当于会将相同key的数据拉取到一个shuffle read task中再进行join，此时就是reduce join。但是如果一个RDD是比较小的，则可以采用广播小RDD全量数据+map算子来实现与join同样的效果，也就是map join，此时就不会发生shuffle操作，也就不会发生数据倾斜。
## 7. 采样倾斜key并分拆join操作
**方案适用场景：** 两个RDD/Hive表进行join的时候，如果数据量都比较大，无法采用“解决方案五”，那么此时可以看一下两个RDD/Hive表中的key分布情况。如果出现数据倾斜，是因为其中某一个RDD/Hive表中的少数几个key的数据量过大，而另一个RDD/Hive表中的所有key都分布比较均匀，那么采用这个解决方案是比较合适的。    

对于join导致的数据倾斜，如果只是某几个key导致了倾斜，可以将少数几个key分拆成独立RDD，并附加随机前缀打散成n份去进行join，此时这几个key对应的数据就不会集中在少数几个task上，而是分散到多个task进行join了。   

**注意：** 将需要join的**另一个**RDD，也过滤出来那几个倾斜key对应的数据并形成一个单独的RDD，将每条数据**膨胀成n条数据**，这n条数据都按顺序附加一个0~n的前缀，不会导致倾斜的大部分key也形成另外一个RDD。
## 8. 使用随机前缀和扩容RDD进行join
**方案适用场景：** 如果在进行join操作时，RDD中有大量的key导致数据倾斜，那么进行分拆key也没什么意义，此时就只能使用最后一种方案来解决问题了。 

这一种方案是针对有大量倾斜key的情况，将原先一样的key通过附加随机前缀变成不一样的key，然后就可以将这些处理后的“不同key”分散到多个task中去处理，而不是让一个task处理大量的相同key。
## 9. 多种方案组合使用


# 参考文献
[Spark性能优化：数据倾斜调优](https://www.iteblog.com/archives/1671.html)    
[Spark性能优化之道——解决Spark数据倾斜（Data Skew）的N种姿势](http://www.jasongj.com/spark/skew/)    
[四种解决Spark数据倾斜（Data Skew）的方法](https://www.iteblog.com/archives/2061.html)    