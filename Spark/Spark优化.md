* [开发调优](#开发调优)
* [资源调优](#资源调优)
* [数据倾斜调优](#数据倾斜调优)
* [shuffle调优](#shuffle调优)
    * [shuffle相关参数调优](#shuffle相关参数调优)
* [参考文献](#参考文献)

详细内容参见参考文献——美团技术博客中的总结。
# 开发调优
1. 原则一：避免创建重复的RDD
2. 原则二：尽可能复用同一个RDD
3. 原则三：对多次使用的RDD进行持久化
4. 原则四：尽量避免使用shuffle类算子
5. 原则五：使用map-side预聚合的shuffle操作
6. 原则六：使用高性能的算子
    - 使用reduceByKey/aggregateByKey替代groupByKey
    - 使用mapPartitions替代普通map
    - 使用foreachPartitions替代foreach
    - 使用filter之后进行coalesce操作
    - 使用repartitionAndSortWithinPartitions替代repartition与sort类操作
7. 原则七：广播大变量
8. 原则八：使用Kryo优化序列化性能
9. 原则九：优化数据结构

# 资源调优
1. 资源参数调优
    - num-executors
    - executor-memory
    - executor-cores
    - driver-memory
    - spark.default.parallelism
    - spark.storage.memoryFraction
    - spark.shuffle.memoryFraction

# 数据倾斜调优
1. 解决方案一：使用Hive ETL预处理数据
2. 解决方案二：过滤少数导致倾斜的key
3. 解决方案三：提高shuffle操作的并行度
4. 解决方案四：两阶段聚合（局部聚合+全局聚合）
5. 解决方案五：将reduce join转为map join
6. 解决方案六：采样倾斜key并分拆join操作
7. 解决方案七：使用随机前缀和扩容RDD进行join
8. 解决方案八：多种方案组合使用

# shuffle调优
## shuffle相关参数调优
1. spark.shuffle.file.buffer
2. spark.reducer.maxSizeInFlight
3. spark.shuffle.io.maxRetries
4. spark.shuffle.io.retryWait
5. spark.shuffle.memoryFraction
6. spark.shuffle.manager
7. spark.shuffle.sort.bypassMergeThreshold
8. spark.shuffle.consolidateFiles


# 参考文献
[Spark性能优化指南——基础篇](https://tech.meituan.com/spark_tuning_basic.html)    
[Spark性能优化指南——高级篇](https://tech.meituan.com/spark_tuning_pro.html)   