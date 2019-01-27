<!-- GFM-TOC -->
* [HDFS架构](#HDFS架构)
* [HDFS写数据流程](#HDFS写数据流程)
* [HDFS读数据流程](#HDFS读数据流程)
* [DataNode工作机制](#DataNode工作机制)
* [HDFS的特点](#HDFS的特点)
    * [优点](#优点)
    * [缺点](#缺点)
* [HDFS数据存放和健壮性](#HDFS数据存放和健壮性)
    * [HDFS副本](#HDFS副本)
    * [HDFS数据的健壮性](#HDFS数据的健壮性)
* [Secondary NameNode](#secondary-namenode)
    * [NameNode](#NameNode)
    * [Secondary NameNode](#secondary-namenode-1)
* [参考文献](#参考文献)
<!-- GFM-TOC -->

Hadoop分布式文件系统(HDFS)是分布式计算中数据存储管理的基础。它和现有的分布式文件系统有很多共同点。但同时，它和其他的分布式文件系统的区别也是很明显的。HDFS是一个**高度容错性**的系统，适合部署在廉价的机器上。HDFS能提供高吞吐量的数据访问，非常适合大规模数据集上的应用。   

# HDFS架构
![HDFS架构](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/hdfsarchitecture.png)   

HDFS采用master/slave架构。一个HDFS集群是由一个**Namenode**和一定数目的**Datanodes**组成。
1. Namenode是一个**中心服务器**（master），**负责管理文件系统的名字空间(namespace)和客户端对文件的访问**。Namenode执行文件系统的名字空间操作，比如打开、关闭、重命名文件或目录。它也负责确定数据块到具体Datanode节点的映射。
2. Datanode在集群中**一般是一个节点一个**（slave），**负责管理它所在节点上的存储**。HDFS暴露了文件系统的名字空间，用户能够**以文件的形式**在上面存储数据。从内部看，一个文件其实被分成**一个或多个数据块**，这些块存储在一组Datanode上。Datanode负责处理文件系统客户端的读写请求。在Namenode的统一调度下进行数据块的创建、删除和复制。
3. Client是一个客户端，通过命令可以访问HDFS；与Namenode交互时获取文件的位置信息；与Datanode交互时读取或写入数据；文件写入到HDFS时，Client将文件切分成一个一个的Block，然后存储到不同的Datanode上。

# HDFS写数据流程
1. 客户端向namenode请求上传文件，namenode检查目标文件是否已存在，父目录是否存在。
2. namenode返回是否可以上传
3. 客户端请求第一个block上传到哪几个datanode服务器上
4. namenode返回三个datanode节点，分别为dn1,dn2,dn3
4. 客户端请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成
6. dn1、dn2、dn3逐级应答客户端
7. 客户端开始往dn1上传第一个block，以packet为单位，dn1收到一个packet就会传给dn2，dn2传给dn3，dn1每传一个packet会放入一个应答队列等待应答。
8. 当一个block传输完成之后，客户端再次请求namenode上传第二个block的服务器。（重复执行3-7步）

# HDFS读数据流程
1. 客户端向namenode请求下载文件，namenode通过查询元数据，找到文件块所在的datanode地址
2. 挑选一台datanode（就近原则，然后随机）服务器，请求读取数据
3. datanode开始传输数据给客户端（从磁盘里读取数据放入流，以packet为单位做校验）
4. 客户端以packet单位接收，先缓存在本地，然后写入目标文件中

# DataNode工作机制
1. 一个数据块在datanode上以文件形式存储在磁盘上，包括两个文件，**一个是数据本身，一个是元数据包括数据块长度，块数据校验和以及时间戳**（datanode上管理数据块元数据）。
2. datanode启动后向namenode注册，通过后，周期性的向namenode上报所有的块信息。
3. **心跳是3秒一次**，心跳返回结果带有namenode给该datanode的命令如复制数据块到另一台机器，或删除某个数据块。**如果超过10分钟没有收到某个datanode的心跳，则认为该节点不可用**。
4. 集群运行中可以安全加入和退出一些机器。

# HDFS的特点
## 优点
1. **高容错性**。数据自动保存多个副本。它通过增加副本的形式，提高容错性。某一个副本丢失以后，它可以自动恢复，这是由 HDFS 内部机制实现的。
2. **适合处理高吞吐量**。它是通过移动计算而不是移动数据；它会把数据位置暴露给计算框架；对计算的时延不敏感。
3. **适合存储和管理大规模数据（PB级别）**。
4. **适合简单的一致性**。一次写入，多次读取。文件一旦写入不能修改，只能追加。
5. **适合处理非结构化数据**。

## 缺点
1. **不适合低延时数据访问**。比如毫秒级的来存储数据，这是不行的，它做不到。
2. **不适合小文件存储**。存储大量小文件(这里的小文件是指小于HDFS系统的Block大小的文件（1.0版本默认64M，2.0版本默认128M）)的话，它会占用NameNode大量的内存来存储文件、目录和块信息。这样是不可取的，因为NameNode的内存总是有限的。
3. **不适合并发写入、文件随机修改**。一个文件只能有一个写，不允许多个线程同时写。仅支持数据 append（追加），不支持文件的随机修改。

# HDFS数据存放和健壮性
## HDFS副本
HDFS被设计成能够在一个大集群中跨机器可靠地存储超大文件。它将每个文件存储成一系列的数据块，除了最后一个，所有的数据块都是同样大小的。为了容错，**文件的所有数据块都会有副本**。每个文件的数据块大小和副本系数都是可配置的。应用程序可以指定某个文件的副本数目。副本系数可以在文件创建的时候指定，也可以在之后改变。**HDFS中的文件都是一次性写入的，并且严格要求在任何时候只能有一个写入者。**

**Namenode全权管理数据块的复制，它周期性地从集群中的每个Datanode接收心跳信号和块状态报告(Blockreport)**。接收到心跳信号意味着该Datanode节点工作正常。块状态报告包含了一个该Datanode上所有数据块的列表。   

![数据copy](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/hdfsdatanodes.png)   

**默认副本系数是3**，**HDFS的存放策略**是：
- 将一个副本存放在本地机架的节点上；
- 一个副本放在同一机架的另一个节点上
- 最后一个副本放在不同机架的节点上。
 
![HDFS-replica-placement v1](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/HDFS-replica-placement%20v1.jpg)

**优化后的存放策略**是：
- 如果写请求方所在机器是其中一个datanode,则直接存放在本地,否则随机在集群中选择一个datanode；
- 第二个副本存放于不同第一个副本的所在的机架；
- 第三个副本存放于第二个副本所在的机架,但是属于不同的节点。

![HDFS-replica-placement](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/HDFS-replica-placement.jpg)   

## HDFS数据的健壮性
HDFS的主要目标就是即使在出错的情况下也要保证数据存储的可靠性。常见的三种出错情况是：Namenode出错, Datanode出错和网络割裂(network partitions)。   
- 磁盘数据错误，心跳检测和重新复制。    
    每个Datanode节点周期性地向Namenode发送**心跳信号**。Namenode将近期不再发送心跳信号Datanode标记为**宕机**，Namenode不断地检测数据块的副本系数是否低于指定值，一旦发现就启动**复制操作**。在下列情况下，可能需要重新复制：某个Datanode节点失效，某个副本遭到损坏，Datanode上的硬盘错误，或者文件的副本系数增大。
- 集群均衡。   
    **HDFS的架构支持数据均衡策略**。如果某个Datanode节点上的空闲空间低于特定的临界点，按照均衡策略系统就会自动地将数据从这个Datanode移动到其他空闲的Datanode。
- 数据完整性。   
    **HDFS客户端软件实现了对HDFS文件内容的校验和(checksum)检查**。当客户端创建一个新的HDFS文件，会计算这个文件每个数据块的校验和，并将校验和作为一个单独的隐藏文件保存在同一个HDFS名字空间下。当客户端获取文件内容后，它会检验从Datanode获取的数据跟相应的校验和文件中的校验和是否匹配，如果不匹配，客户端可以选择从其他Datanode获取该数据块的副本。
- 元数据磁盘错误。   
    **FsImage和Editlog是HDFS的核心数据结构**。Namenode可以配置成支持维护多个FsImage和Editlog的副本。任何对FsImage或者Editlog的修改，都将同步到它们的副本上。当Namenode重启的时候，它会选取最近的完整的FsImage和Editlog来使用。
- 快照。
    快照支持某一特定时刻的数据的复制备份。**利用快照，可以让HDFS在数据损坏时恢复到过去一个已知正确的时间点**。

# Secondary NameNode
## NameNode
Hadoop NameNode 管理着文件系统的 **Namespace**，它维护着整个**文件系统树**（FileSystem Tree）以及文件树中所有的**文件和文件夹元数据**（Metadata）。这些信息是存在内存中的，但是这些信息也可以持久化到磁盘上。

![namenode操作](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/namenode.png)

NameNode Metadata 主要是两个文件：`Edit log`  和 `fs\_image` 。
- `fs\_image`：它是在NameNode启动时对整个文件系统的快照，是 HDFS 的最新状态（截止到 `fs\_image` 文件创建时间的最新状态）文件；
- `Edit log`：是自 `fs\_image` 创建后的 Namespace 操作日志。

NameNode 每次启动的时候，才会合并两个文件，按照 `Edit log` 的记录，把 `fs\_image` 文件更新到最新，从而得到一个文件系统的最新快照。这也意味着当NameNode运行了很长时间后，`Edit log`文件会变得很大。在这种情况下就会出现下面一些问题：
- edit logs文件会变的很大，怎么去管理这个文件是一个挑战。
- NameNode的重启会花费很长时间，因为在`Edit log`中有很多改动要合并到`fs\_image`文件上。
- 如果NameNode挂掉了，那我们就丢失了部分改动记录。（在这个情况下丢失的改动不会很多, 因为丢失的改动应该是还在内存中但是没有写到edit logs的这部分）

## Secondary NameNode
SecondaryNameNode就是来帮助解决上述问题的，它的职责是合并NameNode的`Edit log`到`fs\_image`文件中。   

![secondarynamenode](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/secondarynamenode.png)

1. Secondary NameNode 定期地从 NameNode 上获取元数据；
2. 当它准备获取元数据的时候，就通知 NameNode 暂停写入 `edits` 文件；
3. NameNode收到请求后停止写入 `edits` 文件，之后的 log 记录写入一个名为 `edits.new` 的文件；
4. Secondary NameNode 获取到元数据以后，把 `edits` 文件和 `fsimage` 文件在本机进行合并，创建出一个新的 `fsimage` 文件；
5. 然后把新的 `fsimage` 文件发送回 NameNode；
6. NameNode 收到 Secondary NameNode 发回的 `fsimage` 后，就拿它覆盖掉原来的 `fsimage` 文件，并删除 `edits` 文件，把 `edits.new` 重命名为 `edits`。

Secondary NameNode的整个目的是在HDFS中提供一个检查点。它只是NameNode的一个助手节点。

# 参考文献
[Hadoop分布式文件系统：架构和设计——官方文档 0.18](http://hadoop.apache.org/docs/r1.0.4/cn/hdfs_design.html#%E5%89%AF%E6%9C%AC%E5%AD%98%E6%94%BE%3A+%E6%9C%80%E6%9C%80%E5%BC%80%E5%A7%8B%E7%9A%84%E4%B8%80%E6%AD%A5)   
[HDFS Architecture——官方文档 2.9.1](http://hadoop.apache.org/docs/r2.9.1/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html)   
[HDFS写文件过程分析](http://shiyanjun.cn/archives/942.html)    
[Hadoop分布式文件系统HDFS架构](http://blog.51cto.com/balich/2058317)   
[What is the difference between a standby NameNodes and a secondary NameNode? Does the new Hadoop with YARN have a secondary NameNode?](https://www.quora.com/What-is-the-difference-between-a-standby-NameNodes-and-a-secondary-NameNode-Does-the-new-Hadoop-with-YARN-have-a-secondary-NameNode)   
[[翻译]Secondary NameNode:它究竟有什么作用？](https://www.jianshu.com/p/5d292a9a8c86)    
[浅谈 Hadoop NameNode、SecondaryNameNode、CheckPoint Node、BackupNode](http://zhang-jc.github.io/2017/01/08/%E6%B5%85%E8%B0%88-Hadoop-NameNode%E3%80%81SecondaryNameNode%E3%80%81CheckPoint-Node%E3%80%81BackupNode/)    
[Hadoop HDFS 详解](http://zheming.wang/blog/2015/07/24/17505A21-0204-48AB-8EBE-EAC911B22821/)    
