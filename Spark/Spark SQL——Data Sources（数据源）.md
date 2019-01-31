* [通用 加载/保存 功能](#通用-加载保存-功能)
* [手动指定选项](#手动指定选项)
* [直接运行SQL](#直接运行sql)
* [Save Modes（保存模式）](#save-modes保存模式)
* [保存到持久表](#保存到持久表)
    * [分桶, 排序和分区](#分桶-排序和分区)

参见[Data Sources （数据源）](http://spark.apachecn.org/docs/cn/2.2.0/sql-programming-guide.html#data-sources-%E6%95%B0%E6%8D%AE%E6%BA%90)   

# 通用 加载/保存 功能
- **spark.read().load(filepath)**
- **df.write().save(filename)**

```java
Dataset<Row> usersDF = spark.read().load("examples/src/main/resources/users.parquet");

usersDF.select("name", "favorite_color").write().save("namesAndFavColors.parquet");
```

# 手动指定选项
- 可以使用**format()**，通过指定文件格式完成对应文件格式的文件的读取和保存。
- 主要的文件格式包含：json, parquet, jdbc, orc, libsvm, csv, text

```java
Dataset<Row> peopleDF = spark.read().format("json").load("examples/src/main/resources/people.json");

peopleDF.select("name", "age").write().format("parquet").save("namesAndAges.parquet");
```

# 直接运行SQL
可以直接用 SQL 将查询结果加载到 DataFrame，该SQL也可以直接对文件操作。

```
Dataset<Row> sqlDF = spark.sql("SELECT * FROM parquet.`examples/src/main/resources/users.parquet`");
```

# Save Modes（保存模式）
Save operations （保存操作）可以选择使用 **SaveMode** , 它指定如何处理现有数据如果存在的话. 重要的是要意识到, 这些 save modes （保存模式）不使用任何 locking （锁定）并且不是 atomic （原子）. 另外, 当执行 Overwrite 时, 数据将在新数据写出之前被删除.    

![savemode](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/savemode.png)

# 保存到持久表
DataFrames 也可以使用saveAsTable()将dataframe持久化保存到Hive中. 作为**内部表**保存。

## 分桶, 排序和分区

```java
peopleDF.write().bucketBy(42, "name").sortBy("age").saveAsTable("people_bucketed");

usersDF
  .write()
  .partitionBy("favorite_color")
  .format("parquet")
  .save("namesPartByColor.parquet");
  
peopleDF
  .write()
  .partitionBy("favorite_color")
  .bucketBy(42, "name")
  .saveAsTable("people_partitioned_bucketed");
```

