* [预先分区](#预先分区)
* [Rowkey优化](#预先分区)
* [减少列族数量](#预先分区)
* [缓存策略](#预先分区)
* [参考文献](#预先分区)


# 预先分区
一种可以加快批量写入速度的方法是通过**预先创建一些空的 Regions**，这样当数据写入 HBase 时，会按照 Region 分区情况，在集群内做数据的负载均衡。

# Rowkey优化
参看[HBase学习之五:HBase的RowKey设计原则](https://blog.csdn.net/javajxz008/article/details/51892967)   
- rowkey长度原则：建议越短越好，不要超过16个字节。原因如下：
    1. 数据的持久化文件HFile中是按照KeyValue存储的，如果rowkey过长，比如超过100字节，1000w行数据，光rowkey就要占用100\*1000w=10亿个字节，将近1G数据，这样**会极大影响HFile的存储效率**；
    2. **MemStore将缓存部分数据到内存，如果rowkey字段过长，内存的有效利用率就会降低，系统不能缓存更多的数据**，这样**会降低检索效率**。
    3. 目前操作系统都是64位系统，内存8字节对齐，控制在16个字节，8字节的整数倍利用了操作系统的最佳特性。
- rowkey散列原则：建议将rowkey的**高位作为散列字段，由程序随机生成，低位放时间字段**，这样将**提高数据均衡分布在每个RegionServer**，以实现负载均衡的几率。避免产生热点问题。如果没有散列字段，首字段直接是时间信息将产生所有新数据都在一个RegionServer上堆积的热点现象，这样在做数据检索的时候负载将会集中在个别RegionServer，降低查询效率。
- rowkey唯一原则：必须在设计上保证其唯一性，rowkey是按照字典顺序排序存储的。

# 减少列族数量
不要在一张表里定义太多的 ColumnFamily。目前 Hbase 并不能很好的处理超过 2~3 个 ColumnFamily 的表。因为某个 ColumnFamily 在 flush 的时候，它邻近的 ColumnFamily 也会因关联效应被触发 flush，最终导致系统产生更多的 I/O。   
如果一个HRegion中Memstore过多，每次flush的开销必然会很大，因此我们也建议在进行表设计的时候尽量减少ColumnFamily的个数。

# 缓存策略
创建表的时候，可以通过 HColumnDescriptor.setInMemory(true) 将表放到 RegionServer 的缓存中，保证在读取的时候被 cache 命中。

# 参考文献
[Hbase架构与原理](https://www.jianshu.com/p/3832ae37fac4)    
[大数据性能调优之HBASE的ROWKEY设计](http://blog.chedushi.com/archives/9720)   
[HBase最佳实践－列族设计优化](http://hbasefly.com/2016/07/02/hbase-pracise-cfsetting/)