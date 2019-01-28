* [MapReduce结果存储](#mapreduce结果存储)

# MapReduce结果存储
各个map任务运行完之后，**输出写入运行任务的机器磁盘中（本地文件系统中，因为任务完成后就会删除map的输出结果，放到hdfs中会进行分块容错处理，所以没必要放到hdfs中。）**。Reducer需要从各map任务中提取自己的那一部分数据（对应的partition）。每个map任务的完成时间可能是不一样的，**reduce任务在map任务结束之后会尽快取走输出结果**。

数据被reduce提走之后，map机器不会立刻删除数据，这是为了预防reduce任务失败需要重做。因此**map输出数据是在整个作业完成之后才被删除掉的**。

**reduce处理后的结果放在HDFS。**

参见[Hadoop5-Mapreduce shuffle及优化](https://www.jianshu.com/p/d903dca59aac)和[Hadoop MapReduce执行过程详解（带hadoop例子）](https://my.oschina.net/itblog/blog/275294)