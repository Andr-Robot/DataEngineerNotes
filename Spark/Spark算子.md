* [Spark算子分类](#spark算子分类)
* [map和flatmap](#map和flatmap)
* [mapPartiions](#mappartiions)
* [groupBy和groupByKey](#groupby和groupbykey)
* [union、intersection和subtract](#unionintersection和subtract)
* [sample和takeSample](#sample和takesample)
* [cache和persist](#cache和persist)
* [coalesce和repartition](#coalesce和repartition)
* [partitionBy、mapValues和flatMapValues](#partitionbymapvalues和flatmapvalues)
* [combineByKey](#combinebykey)
* [reduce和reduceByKey](#reduce和reducebykey)
* [take和top](#take和top)
* [first、count和collect](#firstcount和collect)
* [countByKey、foreach、foreachPartition和sortBy](#countbykeyforeachforeachpartition和sortby)
* [aggregate](#aggregate)
* [参考文献](#参考文献)

# Spark算子分类
1. Value数据类型的Transformation算子，这种变换不触发提交作业，针对处理的数据项是Value型的数据。
2. Key-Value数据类型的Transformation算子，这种变换不触发提交作业，针对处理的数据项是Key-Value型的数据。
3. Action算子，这类算子会触发SparkContext提交作业。


类型 | 算子
---|---
Value型Transformation算子 | map、flatMap、mapPartiions、groupBy、filter、distinct、sample、takesample、glom、union、cartesian、cache、persist、coalesce、repartition
Key-Value型Transformation算子 | mapValues、combineByKey、reduceByKey、partitionBy、cogroup、join、leftOutJoin、rightOutJoin
Actions算子 | foreach、saveAsTextFile、saveAsObjectFile、collect、collectAsMap、reduceByKeyLocally、lookup、count、top、reduce、fold、aggregate

# map和flatmap
参见[Spark算子：RDD基本转换操作(1)–map、flatMap、distinct](http://lxw1234.com/archives/2015/06/325.htm)
- map：将一个RDD中的每个数据项，通过map中的函数映射变为一个新的元素。
- flatmap：第一步和map一样，最后将所有的输出分区合并成一个。

# mapPartiions
参见[Spark算子：RDD基本转换操作(5)–mapPartitions](http://lxw1234.com/archives/2015/07/348.htm)    
该函数和map函数类似，只不过映射函数的参数由RDD中的每一个元素变成了RDD中每一个分区的迭代器。假设有N个元素，有M个分区，那么map的函数的将被调用N次,而mapPartitions被调用M次,**当在映射的过程中不断的创建对象时，使用mapPartitions比map的效率要高很多**。     
比如，将RDD中的所有数据通过JDBC连接写入数据库，如果使用map函数，可能要为每一个元素都创建一个connection，这样开销很大，如果使用mapPartitions，那么只需要针对每一个分区建立一个connection。

# groupBy和groupByKey
参见[Spark函数之groupBy和groupByKey](http://blog.cheyo.net/178.html)
- groupBy：将元素通过函数生成相应的Key，数据就转化为Key-Value格式，之后将Key相同的元素分为一组。
- groupByKey：对Key-Value形式的RDD的操作。与groupBy类似。但是其分组所用的key不是由指定的函数生成的，而是采用元素本身中的key。

# union、intersection和subtract
参见[Spark算子：RDD基本转换操作(4)–union、intersection、subtract](http://lxw1234.com/archives/2015/07/345.htm)
- union：就是将两个RDD进行合并，不去重。
- intersection：该函数返回两个RDD的交集，并且去重。
- subtract：返回在RDD中出现，并且不在otherRDD中出现的元素，不去重。

# sample和takeSample
参见[Spark算子[15]：sample、takeSample 源码实例详解](https://blog.csdn.net/leen0304/article/details/78818743)
- sample：以指定的随机种子随机抽样出数量为fraction的数据，withReplacement表示是抽出的数据是否放回，true为有放回的抽样，false为无放回的抽样。
- takeSample：takeSample()函数和sample函数是一个原理，但是不使用相对比例采样，而是按设定的采样个数进行采样，同时返回结果不再是RDD，而是相当于对采样后的数据进行collect()，返回结果的集合为单机的数组。该方法**仅在预期结果数组很小的情况下使用**，因为所有数据都被加载到driver的内存中。

# cache和persist
参见[Spark源码之persist方法，cache方法以及StorageLevel](https://blog.csdn.net/do_yourself_go_on/article/details/74739260)    
cache和persist都是**用于将一个RDD进行缓存**，这样在之后使用的过程中就不需要重新计算了，可以大大节省程序运行时间。   
- cache：cache方法实际上调用了无参数的persist方法 
- persist：可以根据情况设置缓存级别，默认存储级别是MEMORY_ONLY

# coalesce和repartition
参见[Spark算子：RDD基本转换操作(2)–coalesce、repartition](http://lxw1234.com/archives/2015/07/341.htm)   
- coalesce：
    ```scala
    def coalesce(numPartitions: Int, shuffle: Boolean = false)(implicit ord: Ordering[T] = null): RDD[T]
    ```
    - 该函数用于将RDD进行重分区，使用HashPartitioner；
    - 第一个参数为重分区的数目，第二个为是否进行shuffle，默认为false。
- repartition：
    
    ```scala
    def repartition(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T]
    ```
    - 该函数其实就是coalesce函数第二个参数为true的实现。

# partitionBy、mapValues和flatMapValues
参见[Spark算子：RDD键值转换操作(1)–partitionBy、mapValues、flatMapValues](http://lxw1234.com/archives/2015/07/356.htm)
- partitionBy:该函数根据partitioner函数生成新的ShuffleRDD，将原RDD重新分区。

    ```scala
    def partitionBy(partitioner: Partitioner): RDD[(K, V)]
    ```
- mapValues：同基本转换操作中的map，只不过mapValues是针对[K,V]中的V值进行map操作。

    ```scala
    def mapValues[U](f: (V) => U): RDD[(K, U)]
    ```
- flatMapValues：同基本转换操作中的flatMap，只不过flatMapValues是针对[K,V]中的V值进行flatMap操作。

    ```scala
    def flatMapValues[U](f: (V) => TraversableOnce[U]): RDD[(K, U)]
    ```

# combineByKey
参见[Spark算子：RDD键值转换操作(2)–combineByKey、foldByKey](http://lxw1234.com/archives/2015/07/358.htm)   
使用用户设置好的聚合函数对每个Key中的Value进行组合(combine)。可以将输入类型为RDD[(K, V)]转成成RDD[(K, C)]

# reduce和reduceByKey
- reduce：根据映射函数f，对RDD中的元素进行二元计算，聚集数据集中的所有元素，然后返回计算结果。这是一个Action算子。

    ```scala
    def reduce(f: (T, T) ⇒ T): T
    // 实例
    val c = sc.parallelize(1 to 10)
    c.reduce((x, y) => x + y) // 结果55
    ```
- reduceByKey：该函数用于将RDD[K,V]中每个K对应的V值根据映射函数来运算。

    ```scala
    def reduceByKey(func: (V, V) => V): RDD[(K, V)]
    // 实例
    val a = sc.parallelize(List((1,2),(1,3),(3,4),(3,6)))
    a.reduceByKey((x,y) => x + y).collect  // 结果 Array((1,5), (3,10))
    ```
    reduceByKey对每个键执行reduce，结果生成RDD; 它不是"action"操作，而是返回ShuffleRDD，是"transformation"。在一个（K，V)对的数据集上调用时，返回一个（K，V）对的数据集，使用指定的reduce函数，将相同key的值聚合到一起。

# take和top
- take用于获取RDD中从0到num-1下标的元素，不排序。

    ```
    def take(num: Int): Array[T]
    ```
- top函数用于从RDD中，按照默认（降序）或者指定的排序规则，返回前num个元素。

    ```
    def top(num: Int)(implicit ord: Ordering[T]): Array[T]
    ```

# first、count和collect
- first：返回RDD中的第一个元素，不排序。
- count：返回RDD中的元素数量。
- collect：用于将一个RDD转换成数组。

# countByKey、foreach、foreachPartition和sortBy
- countByKey：用于统计RDD[K,V]中每个K的数量。
- foreach：用于遍历RDD,将函数f应用于每一个元素。
- foreachPartition：和foreach类似，只不过是对每一个分区使用f。
- sortBy：根据给定的排序k函数将RDD中的元素进行排序。

# aggregate
参见[Spark算子：RDD行动Action操作(3)–aggregate、fold、lookup](http://lxw1234.com/archives/2015/07/394.htm)
```
def aggregate[U](zeroValue: U)(seqOp: (U, T) ⇒ U, combOp: (U, U) ⇒ U)(implicit arg0: ClassTag[U]): U
```

aggregate：用户聚合RDD中的元素，先使用seqOp将RDD中每个分区中的T类型元素聚合成U类型，再使用combOp将之前每个分区聚合后的U类型聚合成U类型，特别注意seqOp和combOp都会使用zeroValue的值，zeroValue的类型为U。

# 参考文献
[Spark学习笔记——Spark算子总结及案例](https://andone1cc.github.io/2017/03/05/Spark/spark%E7%AE%97%E5%AD%90/)   
[Spark算子分类及功能描述](https://blog.csdn.net/silentwolfyh/article/details/72625676)   
[Spark算子系列文章](http://lxw1234.com/archives/2015/07/363.htm)