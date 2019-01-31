* [DAGScheduler](#dagscheduler)
* [TaskScheduler](#taskscheduler)
* [参考文献](#参考文献)

# DAGScheduler
DAGScheduler把一个Spark作业转换成Stage的DAG（Directed Acyclic Graph有向无环图），根据RDD和Stage之间的关系找出开销最小的调度方法，然后把Stage以TaskSet的形式提交给TaskScheduler，下图展示了DAGScheduler的作用：   

![DAGScheduler](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/DAGScheduler.jpg)

# TaskScheduler
DAGScheduler决定了运行Task的理想位置，并把这些信息传递给下层的TaskScheduler。此外，DAGScheduler还处理由于Shuffle数据丢失导致的失败，这有可能需要重新提交运行之前的Stage（非Shuffle数据丢失导致的Task失败由TaskScheduler处理）。    

TaskScheduler维护所有TaskSet，当Executor向Driver发送心跳时，TaskScheduler会根据其资源剩余情况分配相应的Task。另外TaskScheduler还维护着所有Task的运行状态，重试失败的Task。下图展示了TaskScheduler的作用：    

![TaskScheduler](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/TaskScheduler.jpg)    

在不同运行模式中任务调度器具体为：   
1. Spark on Standalone模式为TaskScheduler；
2. YARN-Client模式为YarnClientClusterScheduler
3. YARN-Cluster模式为YarnClusterScheduler


# 参考文献
[Spark计算过程分析](https://yq.aliyun.com/articles/64839)    
[Spark的任务调度](https://blog.csdn.net/pelick/article/details/41866845)    