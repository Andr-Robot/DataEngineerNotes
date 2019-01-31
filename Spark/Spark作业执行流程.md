* [Spark运行基本流程](#spark运行基本流程)
* [启动方式](#启动方式)
* [基于Standalone的Spark](#基于standalone的spark)
  * [集群启动流程](#集群启动流程)
  * [Spark on Standalone运行过程](#spark-on-standalone运行过程)
  * [不同启动方式下的作业执行流程](#不同启动方式下的作业执行流程)
    * [Client模式：Driver运行在集群外的客户端上](#client模式driver运行在集群外的客户端上)
    * [Cluster模式：Driver运行在Worker上](#cluster模式driver运行在worker上)
* [基于YARN的Spark](#基于yarn的spark)
  * [客户端模式 yarn-client：驱动器进程是在集群之外客户端运行](#客户端模式-yarn-client驱动器进程是在集群之外客户端运行)
  * [集群模式 yarn-cluster：驱动器进程是在集群上工作节点运行](#集群模式-yarn-cluster驱动器进程是在集群上工作节点运行)
  * [Spark:Yarn-cluster和Yarn-client区别与联系](#sparkyarn-cluster和yarn-client区别与联系)
* [Driver向Master注册Application过程](#driver向master注册application过程)
* [Driver的任务提交过程](#driver的任务提交过程)
* [参考文献](#参考文献)

Spark集群在设计的时候，并没有在资源管理的设计上对外封闭，而是充分考虑了未来对接一些更强大的资源管理系统，如YARN、Mesos等，所以Spark架构设计将资源管理单独抽象出一层，通过这种抽象能够构建一种适合企业当前技术栈的插件式资源管理模块，从而为不同的计算场景提供不同的资源分配与调度策略。Spark集群模式架构，如下图所示：   
![Spark集群模式架构](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/spark-cluster-overview.png)   

上图中，Spark集群Cluster Manager目前支持如下三种模式：Standalone模式、YARN模式和Mesos模式。这里重要介绍前两种。
- **Standalone模式**：是Spark内部默认实现的一种集群管理模式，这种模式是通过集群中的**Master**来**统一管理资源**，而与Master进行资源请求协商的是Driver内部的StandaloneSchedulerBackend（实际上是其内部的StandaloneAppClient真正与Master通信）。   
- **YARN模式**：YARN模式下，可以将资源的管理统一交给YARN集群的ResourceManager去管理。

# Spark运行基本流程
具体来说，以**SparkContext为程序运行的总入口**，在SparkContext的初始化过程中，Spark会分别创建**DAGScheduler**作业调度和**TaskScheduler**任务调度两级调度模块。   
- **作业调度模块**是基于任务阶段的高层调度模块，它为每个Spark作业计算具有依赖关系的多个调度阶段（通常根据shuffle来划分），然后为每个阶段构建出一组具体的任务（通常会考虑数据的本地性等），然后以**TaskSets（任务组）** 的形式提交给任务调度模块来具体执行。
- **任务调度模块**则负责具体启动任务、监控和汇报任务运行情况。   

![Spark运行基本流程](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/o_sparkProcessDetail.png)    

详细的运行流程为：
1. 构建Spark Application的运行环境（**启动SparkContext**），SparkContext向资源管理器（可以是Standalone、Mesos或YARN）注册并申请运行Executor资源；
2. 资源管理器分配Executor资源并启动StandaloneExecutorBackend(这是在standalone模式下)，Executor运行情况将随着心跳发送到资源管理器上；
3. SparkContext构建成DAG图，将DAG图分解成Stage，并把Taskset发送给Task Scheduler。Executor向SparkContext申请Task，Task Scheduler将Task发放给Executor运行同时SparkContext将应用程序代码发放给Executor。
4. Task在Executor上运行，运行完毕释放所有资源。

Spark运行架构特点：
1. **每个Application获取专属的executor进程，该进程在Application期间一直驻留，并以多线程方式运行tasks**。这种Application隔离机制有其优势的，无论是从调度角度看（每个Driver调度它自己的任务），还是从运行角度看（来自不同Application的Task运行在不同的JVM中）。当然，这也意味着Spark Application不能跨应用程序共享数据，除非将数据写入到外部存储系统。
2. **Spark与资源管理器无关**，只要能够获取executor进程，并能保持相互通信就可以了。
3. **提交SparkContext的Client应该靠近Worker节点**（运行Executor的节点)，最好是在同一个Rack里，因为Spark Application运行过程中SparkContext和Executor之间有大量的信息交换；如果想在远程集群中运行，最好使用RPC将SparkContext提交给集群，不要远离Worker运行SparkContext。
4. **Task采用了数据本地性和推测执行的优化机制**。

# 启动方式
- **客户端模式**：是指 spark 的 driver jvm 进程 是跑着你运行 spark-submit 的那台机器上的， 一般是用于调试和交互，可以即时的看到输出， 启动后， 是不能停掉的当前的 shell 进程的， 不然 driver进程 也就被杀掉了；
- **集群模式**：一般是用于生产环境， 如果用户提交作业后，就可以关掉 client 了， 因为这个时候 driver 是跑在集群中的某一台 node 上的， 所在的客户端已经完成使命了。

# 基于Standalone的Spark
## 集群启动流程
Standalone模式下，Spark集群采用了简单的**Master-Slave**架构模式，Master统一管理所有的Worker，这种模式很常见，我们简单地看下Spark Standalone集群启动的基本流程，如下图所示：   
![Spark Standalone集群启动流程](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/spark-bootstrap-standalone-cluster.png)   

可以看到，Spark集群采用的**消息的模式**进行通信，也就是EDA架构模式，借助于RPC层的优雅设计，任何两个Endpoint想要通信，发送消息并携带数据即可。上图的流程描述如下所示：
1. Master启动时首先创一个RpcEnv对象，负责管理所有通信逻辑
2. Master通过RpcEnv对象创建一个Endpoint，Master就是一个Endpoint，Worker可以与其进行通信
3. Worker启动时也是创一个RpcEnv对象
4. Worker通过RpcEnv对象创建一个Endpoint
5. Worker通过RpcEnv对象，建立到Master的连接，获取到一个RpcEndpointRef对象，通过该对象可以与Master通信
6. Worker向Master注册，注册内容包括主机名、端口、CPU Core数量、内存数量
7. Master接收到Worker的注册，将注册信息维护在内存中的Table中，其中还包含了一个到Worker的RpcEndpointRef对象引用
8. Master回复Worker已经接收到注册，告知Worker已经注册成功
9. 此时如果有用户提交Spark程序，Master需要协调启动Driver；而Worker端收到成功注册响应后，开始周期性向Master发送心跳   

## Spark on Standalone运行过程
**Standalone模式是Spark实现的资源调度框架，其主要的节点有Client节点、Master节点和Worker节点。** 其中Driver既可以运行在Worker节点上中，也可以运行在本地Client端。   

![spark_on_standalone_process](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/spark_on_standalone_process.jpg)    

其运行过程如下：
1. SparkContext连接到Master，向Master注册并申请资源（CPU Core 和Memory）；
2. Master根据SparkContext的资源申请要求和Worker心跳周期内报告的信息决定在哪个Worker上分配资源，然后在该Worker上获取资源，然后启动StandaloneExecutorBackend；
3. StandaloneExecutorBackend向SparkContext注册；
4. SparkContext将Applicaiton代码发送给StandaloneExecutorBackend；并且SparkContext解析Applicaiton代码，构建DAG图，并提交给DAG Scheduler分解成Stage（当碰到Action操作时，就会催生Job；每个Job中含有1个或多个Stage，Stage一般在获取外部数据和shuffle之前产生），然后以Stage（或者称为TaskSet）提交给Task Scheduler，Task Scheduler负责将Task分配到相应的Worker，最后提交给StandaloneExecutorBackend执行；
5. StandaloneExecutorBackend会建立Executor线程池，开始执行Task，并向SparkContext报告，直至Task完成。
6. 所有Task完成后，SparkContext向Master注销，释放资源。

## 不同启动方式下的作业执行流程
### Client模式：Driver运行在集群外的客户端上

```shell
# 独立部署，client模式
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://207.184.161.138:7077 \
  --executor-memory 20G \
  --total-executor-cores 100 \
  /path/to/examples.jar \
  1000
```

![standalone-client](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/standalone-client.png)   

1. 客户端启动作业提交后直接运行用户程序，启动Driver相关的工作：SchedulerBackend、DAGScheduler、TaskScheduler和BlockManagerMaster等。
2. 客户端的Driver向Master注册。
3. Master还会让Worker启动Exeuctor。Worker创建一个ExecutorRunner线程，ExecutorRunner会启动ExecutorBackend进程。
4. ExecutorBackend启动后会向Driver的SchedulerBackend注册。Driver的DAGScheduler解析作业并生成相应的Stage，每个Stage包含的Task通过TaskScheduler分配给Executor执行。
5. 所有stage都完成后作业结束。

### Cluster模式：Driver运行在Worker上

```shell
# 独立部署，cluster模式，异常退出时自动重启
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://207.184.161.138:7077 \
  --deploy-mode cluster
  --supervise
  --executor-memory 20G \
  --total-executor-cores 100 \
  /path/to/examples.jar \
  1000
```

![standalone-cluster](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/standalone-cluster.png)    

1. 客户端提交作业给Master
2. Master让一个Worker启动Driver，Worker创建一个DriverRunner线程，DriverRunner启动SchedulerBackend、DAGScheduler和BlockManagerMaster等。
3. 另外Master还会让其余Worker启动Exeuctor，即ExecutorBackend。Worker创建一个ExecutorRunner线程，ExecutorRunner会启动ExecutorBackend进程。
4. ExecutorBackend启动后会向Driver的SchedulerBackend注册。DAGScheduler会根据用户程序，生成执行计划，并调度执行。对于每个stage的task，都会被存放到TaskScheduler中，ExecutorBackend向SchedulerBackend汇报的时候把TaskScheduler中的task调度到ExecutorBackend执行。
5. 所有stage都完成后作业结束。

# 基于YARN的Spark
![YARN](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/yarn_architecture.gif)   

在 yarn 上面提交运行一个应用的**过程**为：
1. Client 向 YARN 提交应用程序，其中包括 ApplicationMaster 程序及启动 ApplicationMaster 的命令
2. ResourceManager 为该 ApplicationMaster 分配第一个 Container，并与对应的 NodeManager 通信，要求它在这个 Container 中启动应用程序的 ApplicationMaster
3. ApplicationMaster 首先向 ResourceManager 注册
4. ApplicationMaster 为 Application 的任务申请并领取资源，领取到资源后，要求对应的 NodeManager 在 Container 中启动任务。
NodeManager 收到 ApplicationMaster 的请求后，为任务设置好运行环境（包括环境变量、JAR 包、二进制程序等），将任务启动脚本写到一个脚本中，并通过运行该脚本启动任务
5. 各个任务通过 RPC 协议向 ApplicationMaster 汇报自己的状态和进度，以让 ApplicationMaster 随时掌握各个任务的运行状态，从而可以在失败时重启任务
6. 应用程序完成后，ApplicationMaster 向 ResourceManager 注销并关闭自己

无论 spark 以哪种方式启动， 只要是跑在 yarn 上面， 就得遵守yarn 的规矩， 所以 spark 能做的就是按照 yarn 的方式来适配。
## 客户端模式 yarn-client：驱动器进程是在集群之外客户端运行
![yarn-client](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/yarn-client.jpg)    

Yarn-Client模式中，**Driver在客户端本地运行**，这种模式可以使得Spark Application和客户端进行交互，因为Driver在客户端，所以可以通过webUI访问Driver的状态，默认是http://hadoop1:4040访问，而YARN通过http:// hadoop1:8088访问。   

YARN-client的工作流程分为以下几个步骤：
1. Spark Yarn Client向YARN的ResourceManager申请启动Application Master。同时在SparkContext初始化中将创建DAGScheduler和TASKScheduler等，由于我们选择的是Yarn-Client模式，程序会选择**YarnClientClusterScheduler**和**YarnClientSchedulerBackend**；
2. ResourceManager收到请求后，在集群中选择一个NodeManager，为该应用程序分配第一个Container，要求它在这个Container中启动应用程序的ApplicationMaster，与YARN-Cluster区别的是在该ApplicationMaster不运行SparkContext，只与SparkContext进行联系进行资源的分派；
3. Client中的SparkContext初始化完毕后，与ApplicationMaster建立通讯，向ResourceManager注册，根据任务信息向ResourceManager申请资源（Container）；
4. 一旦ApplicationMaster申请到资源（也就是Container）后，便与对应的NodeManager通信，要求它在获得的Container中启动启动CoarseGrainedExecutorBackend，CoarseGrainedExecutorBackend启动后会向Client中的SparkContext注册并申请Task；
5. Client中的SparkContext分配Task给CoarseGrainedExecutorBackend执行，CoarseGrainedExecutorBackend运行Task并向Driver汇报运行的状态和进度，以让Client随时掌握各个任务的运行状态，从而可以在任务失败时重新启动任务；
6. 应用程序运行完成后，Client的SparkContext向ResourceManager申请注销并关闭自己。


## 集群模式 yarn-cluster：驱动器进程是在集群上工作节点运行

```shell
# YARN上运行，cluster模式
export HADOOP_CONF_DIR=XXX
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master yarn \
  --deploy-mode cluster \  # 要client模式就把这个设为client
  --executor-memory 20G \
  --num-executors 50 \
  /path/to/examples.jar \
  1000
```

![yarn-cluster](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/yarn-cluster.jpg)    

在YARN-Cluster模式中，当用户向YARN中提交一个应用程序后，YARN将分两个阶段运行该应用程序：
- 第一个阶段是把Spark的Driver作为一个ApplicationMaster在YARN集群中先启动；
- 第二个阶段是由ApplicationMaster创建应用程序，然后为它向ResourceManager申请资源，并启动Executor来运行Task，同时监控它的整个运行过程，直到运行完成。   

YARN-cluster的工作流程分为以下几个步骤：
1. Spark Yarn Client向YARN中提交应用程序，包括ApplicationMaster程序、启动ApplicationMaster的命令、需要在Executor中运行的程序等；
2. ResourceManager收到请求后，在集群中选择一个NodeManager，为该应用程序分配第一个Container，要求它在这个Container中启动应用程序的ApplicationMaster，其中ApplicationMaster进行SparkContext等的初始化；
3. ApplicationMaster向ResourceManager注册，这样用户可以直接通过ResourceManage查看应用程序的运行状态，然后它将采用轮询的方式通过RPC协议为各个任务申请资源，并监控它们的运行状态直到运行结束；
4. 一旦ApplicationMaster申请到资源（也就是Container）后，便与对应的NodeManager通信，要求它在获得的Container中启动启动CoarseGrainedExecutorBackend，CoarseGrainedExecutorBackend启动后会向ApplicationMaster中的SparkContext注册并申请Task。这一点和Standalone模式一样，只不过SparkContext在Spark Application中初始化时，使用CoarseGrainedSchedulerBackend配合YarnClusterScheduler进行任务的调度，其中YarnClusterScheduler只是对TaskSchedulerImpl的一个简单包装，增加了对Executor的等待逻辑等；
5. ApplicationMaster中的SparkContext分配Task给CoarseGrainedExecutorBackend执行，CoarseGrainedExecutorBackend运行Task并向ApplicationMaster汇报运行的状态和进度，以让ApplicationMaster随时掌握各个任务的运行状态，从而可以在任务失败时重新启动任务；
6. 应用程序运行完成后，ApplicationMaster向ResourceManager申请注销并关闭自己。

## Spark:Yarn-cluster和Yarn-client区别与联系
当在YARN上运行Spark作业，每个Spark executor作为一个YARN容器(container)运行。Spark可以使得多个Tasks在同一个容器(container)里面运行。这是个很大的优点。   
- 从广义上讲，**yarn-cluster适用于生产环境；而yarn-client适用于交互和调试**，也就是希望快速地看到application的输出。
- 从深层次的含义讲，**yarn-cluster和yarn-client模式的区别其实就是Application Master进程的区别**，yarn-cluster模式下，driver运行在AM(Application Master)中，它负责向YARN申请资源，并监督作业的运行状况。当用户提交了作业之后，就可以关掉Client，作业会继续在YARN上运行。然而**yarn-cluster模式不适合运行交互类型的作业**。而yarn-client模式下，Application Master仅仅向YARN请求executor，client会和请求的container通信来调度他们工作，也就是说Client不能离开。

# Driver向Master注册Application过程

```scala
val command = Command("org.apache.spark.executor.CoarseGrainedExecutorBackend", args, sc.executorEnvs, 
　　　　　　　　　　　　classPathEntries, libraryPathEntries, extraJavaOpts)
val sparkHome = sc.getSparkHome()
val appDesc = new ApplicationDescription(sc.appName, maxCores, sc.executorMemory, command,
  　　　　　　　　　　　sparkHome, sc.ui.appUIAddress, sc.eventLogger.map(_.logDir))
client = new AppClient(sc.env.actorSystem, masters, appDesc, this, conf)
client.start()
```

![spark_submit_application](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/spark_submit_application.png)

![Driver向Master注册Application过程](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/driver%E5%90%91master%E6%B3%A8%E5%86%8Capp.png)    

上面的图中涉及到了三方通信，具体的过程如下：
1. Driver通过AppClient向Master发送了RegisterApplication消息来注册Application，Master收到消息之后会发送RegisteredApplication通知Driver注册成功，Driver的接收类还是AppClient。
2. Master接受到RegisterApplication之后会触发调度过程，在资源足够的情况下会向Woker和Driver分别发送LaunchExecutor、ExecutorAdded消息。
3. Worker接收到LaunchExecutor消息之后，会执行消息中携带的命令，执行CoarseGrainedExecutorBackend类(图中仅以它继承的接口ExecutorBackend代替)，执行完毕之后会发送ExecutorStateChanged消息给Master。
4. Master接收ExecutorStateChanged之后，立即发送ExecutorUpdated消息通知Driver。
5. Driver中的AppClient接收到Master发过来的ExecutorAdded和ExecutorUpdated后进行相应的处理。
6. 启动之后的CoarseGrainedExecutorBackend会向Driver发送RegisterExecutor消息。
7. Driver中的SparkDeploySchedulerBackend（具体代码在CoarseGrainedSchedulerBackend里面）接收到RegisterExecutor消息，回复注册成功的消息RegisteredExecutor给ExecutorBackend，并且立马准备给它发送任务。
8. CoarseGrainedExecutorBackend接收到RegisteredExecutor消息之后，实例化一个Executor等待任务的到来

# Driver的任务提交过程
![](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/diverrun.jpg)

1. Driver程序的代码运行到action操作，触发了SparkContext的runJob方法。
2. SparkContext调用DAGScheduler的runJob函数。
3. DAGScheduler把Job划分stage，然后把stage转化为相应的Tasks，把Tasks交给TaskScheduler。
4. 通过TaskScheduler把Tasks添加到任务队列当中，交给SchedulerBackend进行资源分配和任务调度。
5. 调度器给Task分配执行Executor，ExecutorBackend负责执行Task。

# 参考文献
[《Spark官方文档》集群模式概览](http://ifeve.com/%E3%80%8Aspark%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3%E3%80%8B%E9%9B%86%E7%BE%A4%E6%A8%A1%E5%BC%8F%E6%A6%82%E8%A7%88/)   
[Spark Standalone架构设计要点分析](http://shiyanjun.cn/archives/1545.html)   
[Spark 架构与作业执行流程](https://www.jianshu.com/p/e1cf4c58ae35)   
[spark 是怎么跑在 yarn 上面的](http://coolplayer.net/2017/04/26/spark-%E6%98%AF%E6%80%8E%E4%B9%88%E8%B7%91%E5%9C%A8-yarn-%E4%B8%8A%E9%9D%A2%E7%9A%84/)   
[YARN任务提交流程](https://blog.csdn.net/u010039929/article/details/74171927)   
[Spark on YARN客户端模式作业运行全过程分析](https://www.iteblog.com/archives/1191.html)   
[Spark on YARN集群模式作业运行全过程分析](https://www.iteblog.com/archives/1189.html)   
[Spark:Yarn-cluster和Yarn-client区别与联系](https://www.iteblog.com/archives/1223.html)    
[Spark源码系列（四）图解作业生命周期](https://www.cnblogs.com/cenyuhai/p/3801167.html)    
[Spark分布式计算执行模型](http://www.flickering.cn/%E5%88%86%E5%B8%83%E5%BC%8F%E8%AE%A1%E7%AE%97/2014/07/spark%E5%88%86%E5%B8%83%E5%BC%8F%E8%AE%A1%E7%AE%97%E6%89%A7%E8%A1%8C%E6%A8%A1%E5%9E%8B/)   
[Spark基本工作流程及YARN cluster模式原理](https://www.cnblogs.com/BYRans/p/5889374.html)