* [HBase源码分析之KeyValue](#hbase源码分析之keyvalue)
* [删除数据](#删除数据)
* [更新数据](#更新数据)
* [总结](#总结)


# HBase源码分析之KeyValue
参见[HBase源码分析之KeyValue](https://blog.csdn.net/lipeng_bigdata/article/details/51013502)和[HBase的基础类型KeyValue](http://yangbolin.cn/2014/07/20/hbase-keyvalue-type/)    
HBase是**面向列的存储数据**的，最终的存储单元都是**KeyValue**的结构，HBase本身也定义了一个KeyValue的类型，**这是HBase数据存储的基本类型**。   
从名字来看应该只有两个数据，一个是Key,一个是Value,的确如此，不过这里的Key是多个元素的聚合，有**rowkey,列族，列名，时间戳以及key的类型**。  

![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/hbasekv.jpg)

从左到右，依次为：
1. Key Length：存储Key的长度，占4B；
2. Value Length：存储Value的长度，占4B；
3. Key：由Row Length、Row、Column Family Length、Column Family
    1. Row Length：存储Row的长度，即rowkey的长度，占2B；
    2. Row：存储Row实际内容，即Rowkey，其大小为Row Length；
    3. Column Family Length：存储列簇Column Family的长度，占1B；
    4. Column Family：存储Column Family实际内容，大小为Column Family Length；
    5. Column Qualifier：存储Column Qualifier对应的数据，既然key中其他所有字段的大小都知道了，整个key的大小也知道了，那么这个Column Qualifier大小也是明确的了，无需再存储其length；
    6. Time Stamp：存储时间戳Time Stamp，占8B；
    7. Key Type：存储Key类型Key Type，占1B，Type分为**Put、Delete、DeleteColumn、DeleteFamilyVersion、DeleteFamily**等类型，标记这个KeyValue的类型；
4. Value：存储单元格Cell对应的实际的值Value。

![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/HBase-key-value.png)

# 删除数据
由于HBase底层依赖HDFS，对于HBase删除操作来说，HBase无法在查询到之前的数据并进行修改，**只能顺序读写，追加记录**。那HBase只能追加记录了，为了更新或删除数据，HBase会插入一条一模一样的新的数据，但是**key type会标记成Delete状态**，以**标记该记录被删除**。在读取的时候如果取到了是Delete，而且时间是最新的，那么这条记录肯定是被删掉了。

# 更新数据
进行更新操作的时候，也会**重新插入一条新的数据**来代替在原来数据上修改。新的数据的timestamp会大于老的数据，这样读取的时候，判断时间戳就可以取出最新的数据了。

# 总结
由于HBase这样的删除和更新机制，如果后面没有一个对于过期数据处理的机制，会导致过期数据越来越大，因此后面的**compact操作中的major compact**就顺便将过期的数据删除掉了。    

对于标记为删除的数据，直接删除。对于不同时间戳的多条数据，根据其保存的最大版本数据，删除过期的数据。