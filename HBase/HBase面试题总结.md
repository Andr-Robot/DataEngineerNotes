[toc]

# HBase什么时候触发HFile的合并操作，为什么需要合并HFile
当**文件达到一定数量（默认3）** 就会**触发Compact操作**，将多个HFile合并成一个的HFile文件，**同时进行版本合并和数据删除**。     
为了**优化读过程**，因为小文件较多，合并成一个文件，方便更快的读。

# HBase读写流程

# HBase如何保证数据一致性
强一致性是 HBase 的默认一致性模型。
- 每个值只出现在一个 region。
- 同一时间每个 region 只被分配给一个 region server。
- HBase行事务的实现，使所有行内的操作都是原子操作。所有put操作要么完全成功，要么完全失败。
- 通过任何 API 返回的行的内容总是一个完整的行。

# HRegionServer宕机怎么处理
HMaster通过Zookeeper感知到HRegionServer宕机时，会首先处理遗留的HLog文件，将不同的Region的Log数据进行拆分，分别放到对应的Region中，然后再将失效的Region重新分配。当HRegionServer获取到失效的Region时，首先会处理HLog文件，将HLog中的数据回放到MemStore中，然后flush到StoreFile，完成数据恢复。

参见[深入HBase架构解析（二）](http://www.blogjava.net/DLevin/archive/2015/08/22/426950.html)

当一台HRegionServer宕机时，由于它不再发送Heartbeat给ZooKeeper而被监测到，此时ZooKeeper会通知HMaster，HMaster会检测到哪台HRegionServer宕机，它将宕机的HRegionServer中的HRegion重新分配给其他的HRegionServer，同时HMaster会把宕机的HRegionServer相关的WAL拆分分配给相应的HRegionServer(将拆分出的WAL文件写入对应的目的HRegionServer的WAL目录中，并写入对应的DataNode中），从而这些HRegionServer可以**重放**分到的WAL来重建MemStore。   

![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/HBaseArchitecture-Blog-Fig25.png)    

![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/HBaseArchitecture-Blog-Fig26.png)

# HRegionServer宕机回滚时如果有写数据怎么办
RS宕机回放吧，**在回放完成之前不允许写入**。

当某台region server fail的时候，它管理的region failover（故障转移）到其他region server时，需要根据WAL log（Write-Ahead Logging）来redo(redolog，有一种日志文件叫做重做日志文件)，**这时候进行redo的region应该是unavailable的**，所以hbase**降低了可用性，提高了一致性**。设想一下，如果redo的region能够响应请求，那么可用性提高了，则必然返回不一致的数据(因为redo可能还没完成)，那么hbase就降低一致性来提高可用性了。

# 读blockcache时，如果有删除数据进memstore，怎么保证一致性
HBase会先查memstore中的数据，如果有删除就不会查历史数据了

# memstore多大刷盘
128M

# 小文件怎么合并

# HBase怎么删除数据

# 常说HBase数据读取要读Memstore、HFile和Blockcache，为什么上面Scanner只有StoreFileScanner和MemstoreScanner两种？没有BlockcacheScanner?
HBase中数据仅仅独立地存在于Memstore和StoreFile中，Blockcache中的数据只是StoreFile中的部分数据（热点数据），即**所有存在于Blockcache的数据必然存在于StoreFile中。因此MemstoreScanner和StoreFileScanner就可以覆盖到所有数据**。实际读取时StoreFileScanner通过索引定位到待查找key所在的block之后，首先检查该block是否存在于Blockcache中，如果存在直接取出，如果不存在再到对应的StoreFile中读取。    

# 数据更新操作先将数据写入Memstore，再落盘。落盘之后需不需要更新Blockcache中对应的kv？如果不更新，会不会读到脏数据？
不需要更新Blockcache中对应的kv，而且不会读到脏数据。数据写入Memstore落盘会形成新的文件，和Blockcache里面的数据是相互独立的，以多版本的方式存在。

# 当使用get tablename rowkey的时候，在memstore查询到一条刚刚put的记录后，还会去hregion中继续查找其他timestamp对应的数据吗？
如果只查最新版本，查到第一个版本就返回了，如果是多版本，则先查memstore 再依次查sf。   

# HBase中HRegion的分裂




 
