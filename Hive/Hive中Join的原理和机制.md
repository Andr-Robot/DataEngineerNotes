* [Hive Common Join](#hive-common-join)
    * [Map阶段](#map阶段)
    * [Shuffle阶段](#shuffle阶段)
    * [Reduce阶段](#reduce阶段)
* [Hive Map Join](#hive-map-join)
* [参考文献](#参考文献)

Hive中的Join可分为：**Common Join（Reduce阶段完成join）** 和 **Map Join（Map阶段完成join）**

# Hive Common Join
在Reduce阶段完成join。整个过程包含Map、Shuffle、Reduce阶段。

## Map阶段
- 读取源表的数据，Map输出时候以`Join on`条件中的列为key，如果Join有多个关联键，则以这些关联键的组合作为key;
- Map输出的value为join之后所关心的(select或者where中需要用到的)列；同时在value中还会包含表的Tag信息，用于标明此value对应哪个表；
- 按照key进行排序。

## Shuffle阶段
根据key的值进行hash，并将key/value按照hash值推送至不同的reduce中，这样确保两个表中相同的key位于同一个reduce中。

## Reduce阶段
根据key的值完成join操作，期间通过Tag来识别不同表中的数据。

# Hive Map Join
也叫 broadcast join，join时，**将小表load到每个节点的内存中**，和大表在该节点上的数据进行join，在map端完成join。其中的一个表必须为能完全加载到内存中。这种方式对大表只做单次扫描，速度较快。扫描大表，根据大表寻找与小表关联的数据并输出。

由于**MapJoin没有Reduce**，所以由Map直接输出结果文件，有多少个Map Task，就有多少个结果文件。

# 参考文献
[Hive中Join的原理和机制](http://lxw1234.com/archives/2015/06/313.htm)    
[Hive JOIN使用详解](http://shiyanjun.cn/archives/588.html)    
[Hive:JOIN及JOIN优化](http://datavalley.github.io/2015/10/25/Hive%E4%B9%8BJOIN%E5%8F%8AJOIN%E4%BC%98%E5%8C%96)


