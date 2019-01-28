* [Block](#block)
* [Split](#split)
* [总结](#总结)
* [参考文献](#参考文献)


# Block   
当我们把文件上传到HDFS时，文件会被分块，**这个是真实物理上的划分**。每块的大小可以通过hadoop-default.xml里配置选项进行设置。系统也提供默认大小，其中Hadoop 1.x中的默认大小为**64M**，而Hadoop 2.x中的默认大小为**128M**。每个Block分别存储在多个DataNode上（默认是3个），用于数据备份进而提供数据容错能力和提高可用性。   
在很多分布式文件系统中我们都可以看到Block的存在，这种设计的**优点**是：   
- 存储的文件大小可以大于集群中任意一个磁盘的容量。这很好理解，文件被划分到多个Block中存储，对磁盘透明；
- 使用Block抽象而非整个文件作为存储单元，可以极大简化存储子系统的设计。因为Block size是统一的，因此一个节点上可以存储多少Block就是可以推算的；
- Block 非常适合用于数据备份，进而提供数据容错能力和可用性。    

同样在磁盘中，每个磁盘都有默认的数据块大小，这是磁盘进行数据读/写的最小单位，磁盘块一般为**512字节**。但是在分布式文件系统中数据块一般远大于磁盘数据块的大小，并且为磁盘块大小的整数倍，例如HDFS Block size默认为64MB。   

分布式存储系统中选择大Block size的主要原因是为了**最小化寻址开销**，使得磁盘传输数据的时间可以明显大于定位这个块所需的时间。然而，在HDFS中Block size也不好设置的过大，这是因为MapReduce中的map任务通常一次处理一个块中的数据，因此如果Block太大，则map数就会减少，作业运行的并行度就会受到影响，速度就会较慢。    

如果一个HDFS上的文件大小(file size) 小于块大小(block size) ，那么HDFS会实际占用Linux file system的多大空间？   
**答案**是实际的文件大小，而非一个块的大小。   
# Split
split 是**逻辑意义**上的split。 通常在 M/R 程序或者其他数据处理技术上用到。根据你处理的数据量的情况，split size是允许用户自定义的。    
split size 定义好了之后，可以控制 M/R 中 Mapper 的数量。如果M/R中没有定义 split size ， 就用默认的HDFS配置作为 input split。   
**输入分片（Input Split）**：在进行map计算之前，mapreduce会根据输入文件计算输入分片（input split），每个输入分片（input split）针对一个map任务，输入分片（input split）存储的并非数据本身，而是一个分片长度和一个记录数据的位置的数组。   

**通常一个split就是一个block**（FileInputFormat仅仅拆分比block大的文件），这样做的好处是使得Map可以在存储有当前数据的节点上运行本地的任务，而不需要通过网络进行跨节点的任务调度。   

通过`mapred.min.split.size`， `mapred.max.split.size`, `block.size`来控制拆分的大小。   
- 如果`mapred.min.split.size`**大于**`block size`，则会将两个block合成到一个split，这样有部分block数据需要通过网络读取。   
- 如果`mapred.max.split.size`**小于**`block size`，则会将一个block拆成多个split，增加了Map任务数。   
- 先获取文件在HDFS上的路径和Block信息，然后根据splitSize对文件进行切分（` splitSize = computeSplitSize(blockSize, minSize, maxSize)`），默认splitSize 就等于blockSize的默认值（64m）。   

# 总结
- block是物理上的数据分割，而split是逻辑上的分割。
- 如果没有特别指定，split size 就等于 HDFS 的 block size 。
- 用户可以在M/R 程序中自定义split size。
- 一个split 可以包含多个blocks，也可以把一个block应用多个split操作。
- 有多少个split，就有多少个mapper。   

# 参考文献
[HDFS 块和 Input Splits 的区别与联系](https://mp.weixin.qq.com/s/k8pQ03QvYjQuTF5St49kRg?client=tim&ADUIN=346055491&ADSESSION=1527468725&ADTAG=CLIENT.QQ.5543_.0&ADPUBNO=26767)   
[Split size vs Block size in Hadoop](https://stackoverflow.com/questions/30549261/split-size-vs-block-size-in-hadoop)