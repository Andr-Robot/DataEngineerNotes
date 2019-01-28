* [Hadoop简介]()
    * [是什么](#是什么)
    * [解决了什么问题](#解决了什么问题)
    * [局限和不足](#局限和不足)
    * [Map/Reduce作业]()
    * [流行的计算框架有哪些](#流行的计算框架有哪些)
* [Hadoop基础架构的变化]()
    * [YARN的好处]()
    * [MR v1]()
        * [早期Hadoop架构]()
        * [MR v1架构]()
        * [MapReduce作业处理流程]()
        * [MRv1 的缺陷]()
    * [MR v2(YARN)]()
        * [目前Hadoop架构]()
        * [MR v2(YARN)架构]()
        * [新旧Hadoop MapReduce框架比对]()
        * [MR v2的优势]()
* [参考文献](#参考文献)


# Hadoop简介
## 是什么
Hadoop是一个由Apache基金会所开发的**分布式系统基础架构**，主要由两部分组成：
- 用于存储的文件系统——**HDFS**：在由普通PC组成的集群上提供高可靠的文件存储，通过将**块**保存多个副本的办法解决服务器或硬盘坏掉的问题。
- 用于计算的架构——**MapReduce**：通过简单的**Mapper和Reducer**的抽象提供一个编程模型，可以在一个由几十台上百台的PC组成的可靠集群上**并发地、分布式地**处理大量的数据集，而把并发、分布式（如机器间通信）和故障恢复等计算细节隐藏起来。Mapper和Reducer的抽象，可以将各种各样的复杂数据处理都分解为的基本元素。这样，复杂的数据处理可以分解为由多个Job（包含一个Mapper和一个Reducer）组成的**有向无环图**（DAG）,然后每个Mapper和Reducer放到Hadoop集群上执行，就可以得出结果。

## 解决了什么问题
解决了大数据（大到一台计算机无法进行存储，一台计算机无法在要求的时间内进行处理）的可靠存储和处理。

## 局限和不足
- **抽象层次**低，需要手工编写代码来完成，使用上难以上手。 
- 只提供两个操作，Map和Reduce,**表达力欠缺**。
- 一个Job只有Map和Reduce两个阶段（Phase），复杂的计算需要大量的Job完成，Job之间的依赖关系是由开发者自己管理的。
- 处理逻辑隐藏在代码细节中，没有整体逻辑。
- **中间结果也放在HDFS文件系统中**。
- **ReduceTask需要等待所有MapTask都完成后才可以开始**。
- **时延高**，只适用Batch数据处理，对于交互式数据处理，实时数据处理的支持不够。
- 对于迭代式数据处理性能比较差。

## Map/Reduce作业
一个**Map/Reduce 作业（job）** 通常会把输入的数据集切分为若干独立的**数据块**，由**map任务（task）** 以完全并行的方式处理它们。框架会对map的输出**先进行排序**，然后把结果输入给**reduce任务**。通常**作业的输入和输出都会被存储在文件系统中**。整个框架负责任务的调度和监控，以及重新执行已经失败的任务。   

最简单的 Map/Reduce 应用程序至少包含 3 个部分：**一个 Map 函数、一个 Reduce 函数和一个 main 函数**。**main函数将作业控制和文件输入/输出结合起来**。在这点上，Hadoop 提供了大量的接口和抽象类，从而为Hadoop应用程序开发人员提供许多工具，可用于调试和性能度量等。

Map/Reduce本身就是**用于并行处理大数据集的软件框架**。Map/Reduce的根源是函数式编程中的map和reduce函数。它由两个可能包含有许多实例（许多Map和Reduce）的操作组成。**Map函数接受一组数据并将其转换为一个键/值对列表**，输入域中的每个元素对应一个键/值对。**Reduce函数接受Map函数生成的列表，然后根据它们的键**（为每个键生成一个键/值对）**缩小键/值对列表**。

## 流行的计算框架有哪些
- **MapReduce:**  这个框架人人皆知，它是一种**离线计算框架**，将一个算法抽象成Map和Reduce两个阶段进行处理，非常**适合数据密集型计算**。
- **Spark:** **MapReduce计算框架不适合**（不是不能做，是不适合，效率太低）**迭代计算**（常见于machine learning领域，比如PageRank）**和交互式计算**（data mining领域，比如SQL查询），MapReduce是一种磁盘计算框架，而Spark则是一种**内存计算框架**，它将数据尽可能放到内存中以提高迭代应用和交互式应用的计算效率。
- **Storm:** **MapReduce也不适合进行流式计算、实时分析**，比如广告点击计算等，而Storm则更擅长这种计算、它在实时性要远远好于MapReduce计算框架。

# Hadoop基础架构的变化
先说说MR v1和MR v2，就是MapReduce v1版本和MapReduce v2版本，从Hadoop 0.23.0 版本开始，Hadoop的MapReduce框架完全重构，发生了根本的变化。新的Hadoop MapReduce框架命名为MapReduce V2或者叫**YARN**。   

## YARN的好处
与MR v1相比，YARN不再是一个单纯的计算框架，而是一个**框架管理器**，用户可以将各种各样的计算框架移植到YARN之上，由YARN进行**统一管理和资源分配**。

## MR v1
### 早期Hadoop架构
![Hadoop架构](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/Hadoopframework1.gif)   
![Hadoop架构](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/hadoopframework.gif)   
- **NameNode:** HDFS分发节点，是一个Hadoop集群的总主节点，**它负责文件系统命名空间和客户端的访问控制**，是**单节点**。
- **DataNode:** HDFS数据节点，是Hadoop集群中的存储节点，**负责块的管理**，它表示分布式文件系统，是**多节点**。
- **JobTracker:** **负责作业调度**，首先用户程序 (JobClient) 提交了一个 job，job 的信息会发送到 Job Tracker 中，Job Tracker 是 Map-reduce 框架的中心，他需要与集群中的机器定时通信 (heartbeat), 需要管理哪些程序应该跑在哪些机器上，需要管理所有 job 失败、重启等操作，是**单节点**。
- **TaskTracker:** TaskTracker 是 Map-reduce 集群中每台机器都有的一个部分，他做的事情**主要是监视自己所在机器的资源情况。TaskTracker 同时监视当前机器的 tasks 运行状况（包括启动和监控作业、获取其输出，以及通知 JobTracker 作业完成）**。TaskTracker 需要把这些信息通过 heartbeat 发送给 JobTracker，JobTracker 会搜集这些信息以给新提交的 job 分配运行在哪些机器上，是**多节点**。

### MR v1架构
![MRv1](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/mrv1.png)   

### MapReduce作业处理流程
![mrv1作业处理流程](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/o_hadoop-mapred.jpg)   
1. 当一个客户端向一个 Hadoop 集群发出一个请求时，此请求由 JobTracker 管理。
2. JobTracker 与 NameNode 联合将工作分发到离它所处理的数据尽可能近的位置。
3. JobTracker 将 Map 和 Reduce 任务安排到一个或多个 TaskTracker 上的可用插槽中。
4. TaskTracker 与 DataNode（分布式文件系统）一起对来自 DataNode 的数据执行 Map 和 Reduce 任务。
5. 当 Map 和 Reduce 任务完成时，TaskTracker 会告知 JobTracker，后者确定所有任务何时完成并最终告知客户作业已完成。

### MRv1 的缺陷
- JobTracker 是 Map-reduce 的集中处理点，存在**单点故障**。
- JobTracker 完成了太多的任务，造成了过多的资源消耗，当 map-reduce job 非常多的时候，会**造成很大的内存开销**，潜在来说，也增加了 JobTracker fail 的风险，这也是业界普遍总结出老 Hadoop 的 Map-Reduce 只能支持 4000 节点主机的上限。
- 在 TaskTracker 端，以 map/reduce task 的数目作为资源的表示过于简单，**没有考虑到 cpu/ 内存的占用情况**，如果两个大内存消耗的 task 被调度到了一块，很容易出现 OOM。
- 在 TaskTracker 端，把资源强制划分为 map task slot 和 reduce task slot, 如果当系统中只有 map task 或者只有 reduce task 的时候，会造成资源的浪费，也就是前面提过的**集群资源利用的问题**。
- 源代码层面分析的时候，会发现代码非常的难读，常常因为一个 class 做了太多的事情，代码量达 3000 多行，造成 class 的**任务不清晰，增加 bug 修复和版本维护的难度**。
- 从操作的角度来看，现在的 Hadoop MapReduce 框架在有任何重要的或者不重要的变化 ( 例如 bug 修复，性能提升和特性化 ) 时，都会强制进行系统级别的升级更新。更糟的是，它不管用户的喜好，强制让分布式集群系统的每一个用户端同时更新。这些更新会让用户为了验证他们之前的应用程序是不是适用新的 Hadoop 版本而浪费大量时间。

## MR v2(YARN)
### 目前Hadoop架构
![Hadoop架构](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/Hadoop_2.0_and_YARN_Advantages_over_Hadoop_1.0_4.png)    
- **NameNode**: HDFS分发节点，是一个Hadoop集群的总主节点，它负责文件系统命名空间和客户端的访问控制。
- **DataNode**：HDFS数据节点，是 Hadoop 集群中的存储节点，它表示分布式文件系统。
- **ResourceManager**：MR资源管理。从某种意义上讲它就是一个**纯粹的调度器**，**它在执行过程中不对应用进行监控和状态跟踪**。同样，它也**不能重启**因应用失败或者硬件错误而运行失败的任务。**ResourceManager是基于应用程序对资源的需求进行调度的**；每一个应用程序需要不同类型的资源因此就需要不同的**容器**。资源包括：内存，CPU，磁盘，网络等等。资源管理器**提供一个调度策略的插件**，它负责将集群资源分配给多个队列和应用程序。**调度插件可以基于现有的能力调度和公平调度模型**。
- **NodeManager**：NodeManager是**执行应用程序的容器**，监控应用程序的资源使用情况 (CPU，内存，硬盘，网络 ) 并且向调度器汇报。
- **ApplicationMaster**：**ApplicationMaster是向ResourceManager索要适当的资源容器，运行任务，跟踪应用程序的状态和监控它们的进程，处理任务的失败原因**。每一个应用都会有一个ApplicationMaster，它是一个详细的框架库，它结合从ResourceManager获得的资源和NodeManager协同工作来运行和监控任务。
- **Secondary Namenode**：它的职责是**合并NameNode的edit logs到fs_image文件中，并将合并文件返回给Namenode。然后Namenode将该文件加载到RAM中**。Secondary Namenode不提供故障转移功能，在Namenode挂掉的情况下，Hadoop管理员必须手动从Secondary Namenode恢复数据。
- **Standby Namenode**：在Hadoop 2.0中，随着HA的引入，Hadoop框架中增加了Standby Namenode。备用namenode节点是用来解决Hadoop 1.x中存在的**SPOF（单点故障）** 问题。**Active NameNode 和 Standby NameNode两台 NameNode 形成互备，一台处于 Active 状态，为主 NameNode，另外一台处于 Standby 状态，为备 NameNode，只有主 NameNode 才能对外提供读写服务**。Standby Namenode提供自动故障转移，以防Active Namenode挂掉。

**注意**：启用HA不是强制性的。但是，启用它时，您不能使用Secondary Namenode。因此，要么启用了Secondary Namenode，要么启用了Standby Namenode。   

### MR v2(YARN)架构
![MRv2](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/Architecture-of-YARN.png)   

![MRv2](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/yarn_architecture.gif)   

重构根本的思想是将**JobTracker两个主要的功能分离成单独的组件**，这两个功能是**资源管理和任务调度/监控**。新的**资源管理器全局管理所有应用程序计算资源的分配**，每一个应用的**ApplicationMaster负责相应的调度和协调**。ResourceManager 和每一台机器的NodeManager能够管理用户在那台机器上的进程并能对计算进行组织。   

**Container分为两大类**：
- **运行ApplicationMaster的Container**：这是由ResourceManager（向内部的资源调度器）申请和启动的，用户提交应用程序时，可指定唯一的ApplicationMaster所需的资源；
- **运行各类任务的Container**：这是由ApplicationMaster向ResourceManager申请的，并由ApplicationMaster与NodeManager通信以启动。

### 新旧Hadoop MapReduce框架比对
原框架中核心的 JobTracker 和 TaskTracker 不见了，取而代之的是 ResourceManager, ApplicationMaster 与 NodeManager 三个部分。
- ResourceManager：是一个中心的服务，它做的事情是调度、启动每一个Job所属的ApplicationMaster、另外监控ApplicationMaster的存在情况。**ResourceManager负责作业与资源的调度**。接收JobSubmitter提交的作业，按照作业的上下文(Context)信息，以及从NodeManager收集来的状态信息，启动调度过程，分配一个Container作为App Mstr。
- NodeManager：功能比较专一，**就是负责 Container 状态的维护，并向 RM 保持心跳**。
- ApplicationMaster：**负责一个Job生命周期内的所有工作**，类似老的框架中 JobTracker。但注意每一个Job（不是每一种）都有一个ApplicationMaster，它运行在ResourceManager以外的机器上。   

### MR v2的优势
- 这个设计**大大减小了JobTracker（也就是现在的ResourceManager）的资源消耗**，并且**让监测每一个Job子任务(tasks)状态的程序分布式化**了，更安全、更优美。
- 在新的YARN中，ApplicationMaster是一个可变更的部分，用户可以对不同的编程模型写自己的App Mst，让更多类型的编程模型能够跑在Hadoop集群中，可以参考hadoop YARN 官方配置模板中的mapred-site.xml配置。
- 对于**资源的表示以内存为单位**(在目前版本的YARN中，没有考虑cpu的占用)，比之前以剩余slot数目更合理。
- **老的框架中，JobTracker一个很大的负担就是监控job下的tasks的运行状况，现在，这个部分就扔给ApplicationMaster做了**，而ResourceManager中有一个模块叫做ApplicationsMasters( 注意不是 ApplicationMaster)，它是监测ApplicationMaster的运行状况，如果出问题，会将其在其他机器上重启。
- **Container是YARN为了将来作资源隔离而提出的一个框架**。这一点应该借鉴了Mesos的工作，目前是一个框架，仅仅提供**java虚拟机内存的隔离**,Hadoop团队的设计思路应该后续能支持更多的资源调度和控制，既然资源表示成内存量，那就没有了之前的map slot/reduce slot分开造成集群资源闲置的尴尬情况。


# 参考文献
[Hadoop总结—思维导图](https://github.com/learrn/Hadoop_note)   
[Hadoop Map/Reduce教程——官方文档](http://hadoop.apache.org/docs/r1.0.4/cn/mapred_tutorial.html)   
[What is the difference between a standby NameNodes and a secondary NameNode? Does the new Hadoop with YARN have a secondary NameNode?](https://www.quora.com/What-is-the-difference-between-a-standby-NameNodes-and-a-secondary-NameNode-Does-the-new-Hadoop-with-YARN-have-a-secondary-NameNode)   
[[翻译]Secondary NameNode:它究竟有什么作用？](https://www.jianshu.com/p/5d292a9a8c86)   
[Hadoop NameNode 高可用 (High Availability) 实现解析](https://www.ibm.com/developerworks/cn/opensource/os-cn-hadoop-name-node/index.html)   
[Hadoop学习（二）Hadoop架构及原理](https://birdben.github.io/2016/09/12/Hadoop/Hadoop%E5%AD%A6%E4%B9%A0%EF%BC%88%E4%BA%8C%EF%BC%89Hadoop%E6%9E%B6%E6%9E%84%E5%8F%8A%E5%8E%9F%E7%90%86/)    