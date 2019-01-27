# HBase 压缩策略
Compaction会从一个region的一个store中选择一些hfile文件进行合并。合并说来原理很简单，先从这些待合并的数据文件中读出KeyValues，再按照由小到大排列后写入一个新的文件中。之后，这个新生成的文件就会取代之前待合并的所有文件对外提供服务。HBase根据合并规模将Compaction分为了两类：**Minor Compaction和Major Compaction**：
- Minor Compaction：根据配置策略，自动检查小文件，合并到大文件，从而减少碎片文件，然而并不会立马删除掉旧 HFile 文件。    
    是指选取一些小的、相邻的StoreFile将他们合并成一个更大的StoreFile，在这个过程中不会处理已经Deleted或Expired的Cell。一次Minor Compaction的结果是更少并且更大的StoreFile。Minor Compaction会影响HBase的性能，所以合并文件的数目是有限制的，**默认10个**。
![Minor Compaction](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/minicompaction.png)
- Major Compaction：与上边不同，每个 CF 中，不管有多少个 HFiles 文件，最终都是将 HFiles 合并到一个大的 HFile 中，并且把所有的旧 HFile 文件删除，即CF 与 HFile 最终变成一一对应的关系。   
    是指将所有的StoreFile合并成一个StoreFile，这个过程还会清理三类无意义数据：被删除的数据、TTL过期数据、版本号超过设定版本号的数据。另外，一般情况下，Major Compaction时间会持续比较长，整个过程会消耗大量系统资源，对上层业务有比较大的影响。因此线上业务都会将关闭自动触发Major Compaction功能，改为手动在业务低峰期触发。
![Major Compaction](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/majorcompaction.png)

# 选择合适HFile合并
HBase接下来会再判断是否满足major compaction条件，如果满足，就会选择全部文件进行合并。判断条件有下面三条，只要满足其中一条就会执行major compaction：

1. 用户强制执行major compaction
2. 长时间没有进行compact（CompactionChecker的判断条件2）且候选文件数小于hbase.hstore.compaction.max（默认10）
3. Store中含有Reference文件，Reference文件是split region产生的临时文件，只是简单的引用文件，一般必须在compact过程中删除

如果不满足major compaction条件，就必然为minor compaction，HBase主要有两种minor策略：RatioBasedCompactionPolicy和ExploringCompactionPolicy，下面分别进行介绍：    
- RatioBasedCompactionPolicy    
从老到新逐一扫描所有候选文件，满足其中条件之一便停止扫描：
    - 当前文件大小 < 比它更新的所有文件大小总和 * ratio，其中ratio是一个可变的比例，在高峰期时ratio为1.2，非高峰期为5，也就是非高峰期允许compact更大的文件。那什么时候是高峰期，什么时候是非高峰期呢？用户可以配置参数hbase.offpeak.start.hour和hbase.offpeak.end.hour来设置高峰期
    - 当前所剩候选文件数 <= hbase.store.compaction.min（默认为3）

停止扫描后，待合并文件就选择出来了，即为当前扫描文件+比它更新的所有文件
- ExploringCompactionPolicy
该策略思路基本和RatioBasedCompactionPolicy相同，不同的是，Ratio策略在找到一个合适的文件集合之后就停止扫描了，而Exploring策略会记录下所有合适的文件集合，并在这些文件集合中寻找最优解。最优解可以理解为：待合并文件数最多或者待合并文件数相同的情况下文件大小较小，这样有利于减少compaction带来的IO消耗。

# 执行HFile文件合并
合并流程说起来也简单，主要分为如下几步：
1. 分别读出待合并hfile文件的KV，并顺序写到位于./tmp目录下的临时文件中
2. 将临时文件移动到对应region的数据目录
3. 将compaction的输入文件路径和输出文件路径封装为KV写入WAL日志，并打上compaction标记，最后强制执行sync
4. 将对应region数据目录下的compaction输入文件全部删除

# 参考文献
[HBase Compaction的前生今世－身世之旅](http://hbasefly.com/2016/07/13/hbase-compaction-1/)    
[初涉 HBase](https://leonlibraries.github.io/2017/04/13/%E5%88%9D%E6%B6%89HBase/)