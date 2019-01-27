参见：
- [HBase原理 – 所有Region切分的细节都在这里了](http://hbasefly.com/2017/08/27/hbase-split/)
- [Hadoop HBase中split原理学习](http://forlan.iteye.com/blog/2374049)
- [HBase笔记：Region拆分策略](http://blog.javachen.com/2014/01/16/hbase-region-split-policy.html)   
- [hbase split 源码分析之split策略](https://segmentfault.com/a/1190000015299102)
 


# Region切分触发策略
HBase已经有多达6种切分触发策略。这里只介绍常见的三种：
1. ConstantSizeRegionSplitPolicy：0.94版本前默认切分策略。按固定长度分割region，固定长度取值**优先获取table的”MAX_FILESIZE” 值**，若没有设定该属性，则**采用在hbase-site.xml中配置的hbase.hregion.max.filesize值**，在0.94版本中这个值的缺省值已经被调整为：`10 * 1024 * 1024 * 1024` 也就是**10G**，**当table的某一region中的某一store大小（如果采用压缩的场景，store大小为压缩后的文件大小）超过了预定的最大固定长度时，对该region进行split**。
    - 总结：**切分策略对于大表和小表没有明显的区分**。
2. IncreasingToUpperBoundRegionSplitPolicy：0.94版本~2.0版本默认切分策略。**一个region中最大store大小大于设置阈值就会触发切分**。但是这个阈值并不像ConstantSizeRegionSplitPolicy是一个固定的值，而是会在一定条件下不断调整，**调整规则和region所属表在当前regionserver上的region个数有关系** ：`(#regions) * (#regions) * (#regions) * flush size * 2`，当然阈值并不会无限增大，最大值为用户设置的**MaxRegionFileSize**（`hbase.hregion.max.filesize`为hbase-site中设定的单个region大小，默认10G ）。这种切分策略很好的弥补了ConstantSizeRegionSplitPolicy的短板，能够自适应大表和小表。
    - 总结：这种策略下很多小表会在大集群中产生大量小region，分散在整个集群中。而且在发生region迁移时也可能会触发region分裂。
3. SteppingSplitPolicy：2.0版本默认切分策略。这种切分策略的切分阈值又发生了变化，相比IncreasingToUpperBoundRegionSplitPolicy简单了一些，依然**和待分裂region所属表在当前regionserver上的region个数有关系，如果region个数等于1，切分阈值为`flush size * 2`，否则为MaxRegionFileSize。**
    - 总结：这种切分策略对于大集群中的大表、小表会比IncreasingToUpperBoundRegionSplitPolicy更加友好，小表不会再产生大量的小region。

还有一些其他分裂策略，比如使用**DisableSplitPolicy**:可以禁止region发生分裂；而**KeyPrefixRegionSplitPolicy**，**DelimitedKeyPrefixRegionSplitPolicy**对于切分策略依然依据默认切分策略，但对于切分点有自己的看法。
> KeyPrefixRegionSplitPolicy指定rowkey前缀位数划分region，要求必须让相同的PrefixKey待在一个region中，此种策略比较适合固定前缀的rowkey。当table中没有设置prefix_split_key_policy.prefix_length属性，或prefix_split_key_policy.prefix_length属性不为Integer类型时，指定此策略效果等同与使用IncreasingToUpperBoundRegionSplitPolicy。 

HBase对一个region切分，有几个条件：
1. 如果是用户请求切分，则不管什么情况都可以切分。
2. 如果非用户请求，并且这个region中任意store含有引用文件，则不切分
3. 如果不是用户请求，也没有引用文件，则判断每个store的大小，只要其中有一个大于阀值，则切分。这个阀值在上面已经有说到。

[IncreasingToUpperBoundRegionSplitPolicy.java](https://github.com/apache/hbase/blob/master/hbase-server/src/main/java/org/apache/hadoop/hbase/regionserver/IncreasingToUpperBoundRegionSplitPolicy.java)

```java
@Override
protected boolean shouldSplit() {
    boolean force = region.shouldForceSplit();
    boolean foundABigStore = false;
    // Get count of regions that have the same common table as this.region
    int tableRegionsCount = getCountOfCommonTableRegions();
    // Get size to check
    long sizeToCheck = getSizeToCheck(tableRegionsCount);

    for (HStore store : region.getStores()) {
      // If any of the stores is unable to split (eg they contain reference files)
      // then don't split
      if (!store.canSplit()) {
        return false;
      }

      // Mark if any store is big enough
      long size = store.getSize();
      if (size > sizeToCheck) {
        LOG.debug("ShouldSplit because " + store.getColumnFamilyName() +
          " size=" + StringUtils.humanSize(size) +
          ", sizeToCheck=" + StringUtils.humanSize(sizeToCheck) +
          ", regionsWithCommonTable=" + tableRegionsCount);
        foundABigStore = true;
      }
    }

    return foundABigStore || force;
}
```

[HStore.java](https://github.com/apache/hbase/blob/master/hbase-server/src/main/java/org/apache/hadoop/hbase/regionserver/HStore.java)

```java
@Override
public boolean canSplit() {
    this.lock.readLock().lock();
    try {
      // Not split-able if we find a reference store file present in the store.
      boolean result = !hasReferences();
      if (!result) {
        LOG.trace("Not splittable; has references: {}", this);
      }
      return result;
    } finally {
      this.lock.readLock().unlock();
    }
}
```

# 寻找SplitPoint
region切分策略会触发region切分，切分开始之后的第一件事是寻找切分点－splitpoint。所有默认切分策略，无论是ConstantSizeRegionSplitPolicy、IncreasingToUpperBoundRegionSplitPolicy抑或是SteppingSplitPolicy，对于切分点的定义都是一致的：
> 整个region中**最大store中的最大文件中最中心的一个block的首个rowkey**。如果定位到的rowkey是整个文件的首个rowkey或者最后一个rowkey的话，就认为没有切分点。

# Region核心切分流程
HBase将整个切分过程包装成了一个事务，意图能够保证切分事务的原子性。整个分裂事务过程分为三个阶段：prepare – execute – (rollback) 。

## prepare阶段
在内存中初始化两个子region，具体是生成两个HRegionInfo对象，包含tableName、regionName、startkey、endkey等。同时会生成一个transaction journal（日志），这个对象用来记录切分的进展，具体见rollback阶段。

## execute阶段
切分的核心操作。见下图：

![regionsplit](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/regionsplit.png)     

1. regionserver 更改ZK节点 /region-in-transition 中该region的状态为SPLITING。
2. master通过watch节点/region-in-transition检测到region状态改变，并修改内存中region的状态，在master页面RIT**(region in transition)**模块就可以看到region执行split的状态信息。
3. 在父存储目录下新建临时文件夹.split保存split后的daughter region信息。
4. 关闭parent region：parent region关闭数据写入并触发flush操作，将写入region的数据全部持久化到磁盘。此后短时间内客户端落在父region上的请求都会抛出异常NotServingRegionException。
5. 核心分裂步骤：在.split文件夹下新建两个子文件夹，称之为daughter A、daughter B，并在文件夹中生成reference文件，分别指向父region中对应文件。
6. 父region分裂为两个子region后，将daughter A、daughter B拷贝到HBase根目录下，形成两个新的region。
7.  parent region通知修改 hbase.meta 表后下线，不再提供服务。下线后parent region在meta表中的信息并不会马上删除，而是标注split列、offline列为true，并记录两个子region。

    ![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/20170827125935_73066.png)

8.  开启daughter A、daughter B两个子region。通知修改 hbase.meta 表，正式对外提供服务。

    ![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/20170827130021_69166.png)

## rollback阶段
如果execute阶段出现异常，则执行rollback操作。为了实现回滚，整个切分过程被分为很多子阶段，回滚程序会根据当前进展到哪个子阶段清理对应的垃圾数据。

![regionsplitprogress](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/regionsplitprogress.png)   

# reference文件
execute阶段中的第5步是所有步骤中最核心的一个环节，生成reference文件日志如下所示：
> 2017-08-12 11:53:38,158 DEBUG [StoreOpener-0155388346c3c919d3f05d7188e885e0-1] regionserver.StoreFileInfo: reference 'hdfs://hdfscluster/hbase-rsgroup/data/default/music/0155388346c3c919d3f05d7188e885e0/cf/d24415c4fb44427b8f698143e5c4d9dc.00bb6239169411e4d0ecb6ddfdbacf66' to region=00bb6239169411e4d0ecb6ddfdbacf66 hfile=d24415c4fb44427b8f698143e5c4d9dc。

其中reference文件名为`d24415c4fb44427b8f698143e5c4d9dc.00bb6239169411e4d0ecb6ddfdbacf66`，根据日志可以看到，切分的父region是`00bb6239169411e4d0ecb6ddfdbacf66`，对应的切分文件是`d24415c4fb44427b8f698143e5c4d9dc`如下所示：   

![reference](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/hbasereferencefile.png)

reference文件是一个引用文件，文件内容主要有两部分构成：
1. 是切分点splitkey，
2. 是一个boolean类型的变量（true或者false），true表示该reference文件引用的是父文件的上半部分（top），而false表示引用的是下半部分 （bottom）。

# 总结
整个region**切分过程并没有涉及数据的移动**，所以切分成本本身并不是很高，可以很快完成。**切分后子region的文件实际没有任何用户数据，文件中存储的仅是一些元数据信息－切分点rowkey等**。

## 通过reference文件如何查找数据？
整个流程如下图所示：

![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/referencefinddata.png)   

1. 根据reference文件名（region名+真实文件名）定位到真实数据所在文件路径
2. 根据文件内容，确定扫描的范围，然后进行扫描。

## 父region的数据什么时候会迁移到子region目录？
**子region发生major_compaction时**。我们知道compaction的执行实际上是将store中所有小文件一个KV一个KV从小到大读出来之后再顺序写入一个大文件，完成之后再将小文件删掉，因此compaction本身就需要读取并写入大量数据。**子region执行major_compaction后会将父目录中属于该子region的所有数据读出来并写入子region目录数据文件中**。

## 父region什么时候会被删除？
实际上**HMaster会启动一个线程定期遍历检查所有处于splitting状态的父region，确定检查父region是否可以被清理**。检测线程首先会在meta表中揪出所有split列为true的region，并加载出其分裂后生成的两个子region（meta表中splitA列和splitB列），**只需要检查此两个子region是否还存在引用文件**，如果都不存在引用文件就可以认为该父region对应的文件可以被删除。    

![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/20170827125935_73066.png)



