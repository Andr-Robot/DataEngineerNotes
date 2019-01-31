* [统一 DataFrame and Dataset API](#统一-dataframe-and-dataset-api)
* [SparkSession](#sparksession)
* [参考文献](#参考文献)

Spark 2.0主要聚焦于三个方面：**对标准的SQL支持**、**统一DataFrame和Dataset API**和**提供SparkSession**。下面重点介绍最后两点。
# 统一 DataFrame and Dataset API
在 spark 2.0 中，**把 dataframe 当作是一种特殊的 dataset**，`dataframe = dataset[row]`，把两者统一为 datasets。

# SparkSession
在 spark 2.0 之前，sparkContext 是 Spark应用的入口。除了 sparkContext，还有 sqlContext，StreamingContext，HiveContext 等其他入口。然而到了 spark 2.0 后，因为逐渐要采用 DataFrame 和 DataSet 作为 API 使用，需要一个统一的入口点，所以就诞生了 SparkSession。**本质上，可以简单的把 SparkSession 理解成 sparkContext, sqlContext, StreamingContext, HiveContext 的统一封装**。   



# 参考文献
[Spark 2.0技术预览：更容易、更快速、更智能](https://www.iteblog.com/archives/1668.html)    
[Spark 2.0介绍：SparkSession创建和使用相关API](https://www.iteblog.com/archives/1673.html)    
[『 Spark 』12. Spark 2.0 | 10 个特性介绍](http://litaotao.github.io/spark-2.0-faster-easier-smarter)     
