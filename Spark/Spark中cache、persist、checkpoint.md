* [Spark 持久化](#spark-持久化)
  * [概述](#概述)
  * [Cache](#cache)
  * [Persist](#persist)
* [Checkpoint](#checkpoint)
  * [Checkpoint vs Cache](#checkpoint-vs-cache)
  * [checkpoint 的正确使用姿势](#checkpoint-的正确使用姿势)
  * [rdd.persist(StorageLevel.DISK_ONLY) vs checkpoint()](#rddpersiststorageleveldisk_only-vs-checkpoint)
* [参考文献](#参考文献)

# Spark 持久化
## 概述
Spark 中一个很重要的能力是**将数据 persisting 持久化（或称为 caching 缓存）**，在多个操作间都可以访问这些持久化的数据。当持久化一个 RDD 时，每个节点的其它分区都可以使用 RDD 在内存中进行计算，在该数据上的其他 action 操作将直接使用内存中的数据。这样会让以后的 action 操作计算速度加快（通常运行速度会加速 10 倍）。缓存是迭代算法和快速的交互式使用的重要工具。     

RDD 可以使用 persist() 方法或 cache() 方法进行持久化。数据将会在第一次 action 操作时进行计算，并缓存在节点的内存中。Spark 的缓存具有容错机制，如果一个缓存的 RDD 的某个分区丢失了，Spark 将按照原来的计算过程，自动重新计算并进行缓存。

最后，被缓存的rdd存在于一个running的应用的生命周期内，如果这个应用终止了，那么缓存的rdd也会同时被删除。

## Cache
Cache 与 MEMORY_ONLY 的持久化级别相同，如以下代码所示：

```scala
/**
 * Persist this RDD with the default storage level (`MEMORY_ONLY`).
 */
def persist(): this.type = persist(StorageLevel.MEMORY_ONLY)

/**
 * Persist this RDD with the default storage level (`MEMORY_ONLY`).
 */
def cache(): this.type = persist()
```

## Persist

```
/**
 * Mark this RDD for persisting using the specified level.
 *
 * @param newLevel the target storage level
 * @param allowOverride whether to override any existing level with the new one
 */
private def persist(newLevel: StorageLevel, allowOverride: Boolean): this.type = {
  // TODO: Handle changes of StorageLevel
  if (storageLevel != StorageLevel.NONE && newLevel != storageLevel && !allowOverride) {
    throw new UnsupportedOperationException(
      "Cannot change storage level of an RDD after it was already assigned a level")
  }
  // If this is the first time this RDD is marked for persisting, register it
  // with the SparkContext for cleanups and accounting. Do this only once.
  if (storageLevel == StorageLevel.NONE) {
    sc.cleaner.foreach(_.registerRDDForCleanup(this))
    sc.persistRDD(this)
  }
  storageLevel = newLevel
  this
}

/**
 * Set this RDD's storage level to persist its values across operations after the first time
 * it is computed. This can only be used to assign a new storage level if the RDD does not
 * have a storage level set yet. Local checkpointing is an exception.
 */
def persist(newLevel: StorageLevel): this.type = {
  if (isLocallyCheckpointed) {
    // This means the user previously called localCheckpoint(), which should have already
    // marked this RDD for persisting. Here we should override the old storage level with
    // one that is explicitly requested by the user (after adapting it to use disk).
    persist(LocalRDDCheckpointData.transformStorageLevel(newLevel), allowOverride = true)
  } else {
    persist(newLevel, allowOverride = false)
  }
}

/**
* Persist this RDD with the default storage level (`MEMORY_ONLY`).
*/
def persist(): this.type = persist(StorageLevel.MEMORY_ONLY)
```

将storageLevel设置为persist()函数传进来的存储级别，而且**一旦设置好RDD的存储级别之后就不能再对相同RDD设置别的存储级别**，否则将会出现异常。设置好存储级别在之后除非触发了action操作，否则不会真正地执行缓存操作。

Persist 意味着将计算出的RDD保存在memory或disk中并在需要时重复使用它。有几种不同级别的持久化：

![存储级别](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/storagelevel.png)     

**注意：** 在 shuffle 操作中（例如 reduceByKey），即便是用户没有调用 persist 方法，Spark 也会自动缓存部分中间数据.这么做的目的是，在 shuffle 的过程中某个节点运行失败时，不需要重新计算所有的输入数据。如果用户想多次使用某个 RDD，强烈推荐在该 RDD 上调用 persist 方法.

# Checkpoint
**checkpoint也是个transformation的算子。**

checkpoint（检查点）是很多分布式系统为了**容灾容错**引入的机制，**其实质是将系统运行时的内存数据结构和状态持久化到磁盘上**，在需要的时候通过读取这些数据，重新构造出之前的运行时状态。

Spark中使用检查点来将RDD的执行状态保存下来，在作业失败重试的时候，从检查点中恢复之前已经运行成功的RDD结果，这样就会大大减少重新计算的成本，提高任务恢复效率和执行效率，节省Spark各个计算节点的资源。

## Checkpoint vs Cache
cache 和 checkpoint 是有显著区别的， **缓存把 RDD 计算出来然后放在内存中**， 但是RDD 的依赖链（相当于数据库中的redo 日志）不能丢掉，如果某个节点出现故障并且上面cache 的RDD的部分分区会丢掉，则可以通过依赖链重放计算出来。不同的是， checkpoint 是把 RDD 保存在 HDFS中， 是多副本可靠存储，所以依赖链就可以丢掉了，就斩断了依赖链， 是通过复制实现的高容错。

**注意：** checkpoint 没有使用这种第一次计算得
到就存储的方法，而是等到 job 结束后另外启动专门的 job 去完成 checkpoint 。因为checkpoint是需要把 job 重新从头算一遍， 最好先cache一下， checkpoint就可以直接保存缓存中的 RDD 了， 就不需要重头计算一遍了， 对性能有极大的提升。   

## checkpoint 的正确使用姿势

```
val data = sc.textFile("/tmp/spark/1.data").cache() // 注意要cache 
sc.setCheckpointDir("/tmp/spark/checkpoint")
data.checkpoint  
data.count
```

使用很简单， 就是**设置一下 checkpoint 目录**，然后**在rdd上调用 checkpoint 方法**， action 的时候就对数据进行了 checkpoint

通过查看checkpoint目录，可以看到生成了partition个二进制文件。

## rdd.persist(StorageLevel.DISK_ONLY) vs checkpoint()
其实 Spark 提供了 `rdd.persist(StorageLevel.DISK_ONLY)` 这样的方法，相当于 cache 到磁盘上，这样可以做到 rdd 第一次被计算得到时就存储到磁盘上，但这个 persist 和 checkpoint 有很多不同。
- `rdd.persist(StorageLevel.DISK_ONLY)`虽然可以将 RDD 的 partition 持久化到磁盘，但该 partition 由 blockManager 管理。**一旦 driver program 执行结束**，也就是 executor 所在进程 CoarseGrainedExecutorBackend stop，blockManager 也会 stop，**被 cache 到磁盘上的 RDD 也会被清空**（整个 blockManager 使用的 local 文件夹被删除）。
- 而 checkpoint 将 RDD 持久化到 HDFS 或本地文件夹，**如果不被手动 remove 掉，是一直存在的**，也就是说可以被下一个 driver program 使用，而 cached RDD 不能被其他 dirver program 使用。



# 参考文献
[Spark Programming Guide-RDD Persistence](https://spark.apache.org/docs/2.2.0/rdd-programming-guide.html#rdd-persistence)     
[Spark 编程指南——RDD Persistence（持久化）](http://spark.apachecn.org/docs/cn/2.2.0/rdd-programming-guide.html#rdd-persistence%E6%8C%81%E4%B9%85%E5%8C%96)   
[彻底理解-spark-的checkpoint-机制](http://coolplayer.net/2017/05/04/%E5%BD%BB%E5%BA%95%E7%90%86%E8%A7%A3-spark-%E7%9A%84checkpoint-%E6%9C%BA%E5%88%B6/)     
[Spark2.3源码分析——RDD的checkpoint（检查点）实现](http://cxy7.com/articles/2018/06/18/1529329223648.html)     
[浅析Apache Spark Caching和Checkpointing](https://segmentfault.com/a/1190000004999929)    
[Spark Persist,Cache以及Checkpoint](http://smartsi.club/2018/07/11/spark-base-persist-cache-checkpoint/)

