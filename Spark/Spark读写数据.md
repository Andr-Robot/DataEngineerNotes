* [Spark加载数据](#spark加载数据)
* [Spark将Dataframe写入到Mysql](#spark将dataframe写入到mysql)
    * [Save Modes（保存模式）](#save-modes保存模式)
* [Spark操作MySQL](#spark操作mysql)
    * [连接MySQL](#连接mysql)
    * [写入MySQL](#写入mysql)
* [参考文献](#参考文献)

# Spark加载数据
参见[Generic Load/Save Functions （通用 加载/保存 功能）](http://spark.apachecn.org/docs/cn/2.2.0/sql-programming-guide.html#generic-loadsave-functions-%E9%80%9A%E7%94%A8-%E5%8A%A0%E8%BD%BD%E4%BF%9D%E5%AD%98-%E5%8A%9F%E8%83%BD)     
在最简单的形式中, 默认数据源（parquet, 除非另有配置 spark.sql.sources.default ）将用于所有操作.

```java
Dataset<Row> usersDF = spark.read().load("examples/src/main/resources/users.parquet");
```

您还可以 **manually specify** （手动指定）将与任何你想传递给 data source 的其他选项一起使用的 data source . Data sources 由其 fully qualified name （完全限定名称）（即 org.apache.spark.sql.parquet ）, 但是对于 built-in sources （内置的源）, 你也可以使用它们的 **shortnames** （短名称）（json, parquet, jdbc, orc, libsvm, csv, text）.从任何 data source type （数据源类型）加载 DataFrames 可以使用此 syntax （语法）转换为其他类型.

```java
Dataset<Row> peopleDF = spark.read().format("json").load("examples/src/main/resources/people.json");
```

不使用读取 API 将文件加载到 DataFrame 并进行查询, 也可以直接用 SQL 查询该文件.

```java
Dataset<Row> sqlDF = spark.sql("SELECT * FROM parquet.`examples/src/main/resources/users.parquet`");
```


# Spark将Dataframe写入到Mysql

```java
// 连接 MySQL
Properties connectionProperties = new Properties();
connectionProperties.put("user", USER);
connectionProperties.put("password", PASSWORD);
connectionProperties.put("driver", JDBC_DRIVER);
// 向 MySQL 表中写入数据
df.write().mode(SaveMode.Append).jdbc(URL, TABLE, connectionProperties);
```

## Save Modes（保存模式）
参见[Save Modes（保存模式）](http://spark.apachecn.org/docs/cn/2.2.0/sql-programming-guide.html#save-modes-%E4%BF%9D%E5%AD%98%E6%A8%A1%E5%BC%8F)

Save operations （保存操作）可以选择使用 **SaveMode** , 它指定如何处理现有数据如果存在的话. 重要的是要意识到, 这些 save modes （保存模式）不使用任何 locking （锁定）并且不是 atomic （原子）. 另外, 当执行 Overwrite 时, 数据将在新数据写出之前被删除.    

![savemode](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/savemode.png)

# Spark操作MySQL
## 连接MySQL
使用jdbc()创建连接。

```java
//jdbc.url=jdbc:mysql://localhost:3306/database
String url = "jdbc:mysql://localhost:3306/test";
//查找的表名
String table = "user_test";
//增加数据库的用户名(user)密码(password),指定test数据库的驱动(driver)
Properties connectionProperties = new Properties();
connectionProperties.put("user","root");
connectionProperties.put("password","123456");
connectionProperties.put("driver","com.mysql.jdbc.Driver");

//SparkJdbc读取Postgresql的products表内容
System.out.println("读取test数据库中的user_test表内容");
// 读取表中所有数据
// select的用法：df.select("colA", "colB")
DataFrame jdbcDF = sqlContext.read().jdbc(url,table,connectionProperties).select("*");
//显示数据
jdbcDF.show();
DataFrame rows = sqlContext.read().jdbc(url, fromTable, connectionProperties).where("count < 1000");
```

Spark还提供通过options()的方式来读取数据。

```java
Map<String, String> props = new HashMap<String, String>();
props.put("url", "jdbc:mysql://spark1:3306/testdb");
props.put("user","root");
props.put("password","123456");
//读取第一个表	
props.put("dbtable", "student_infos");
DataFrame studentInfosDF = sqlContext.read().format("jdbc").options(props).load(); 		
//读取第二个表	
props.put("dbtable", "student_scores"); 	
DataFrame studentScoresDF = sqlContext.read().format("jdbc").options(props).load(); 
```

## 写入MySQL

```
// 向 MySQL 表中写入数据
df.write().mode(SaveMode.Append).jdbc(URL, TABLE, connectionProperties);
```



# 参考文献
[Spark使用Java读取mysql数据和保存数据到mysql](https://blog.csdn.net/fengzhimohan/article/details/78471952)    
[Spark SQL, DataFrames and Datasets Guide](http://spark.apachecn.org/docs/cn/2.2.0/sql-programming-guide.html)    
[使用spark读取mysql数据库数据转化为dataframe](http://www.idataskys.com/2017/07/08/%E4%BD%BF%E7%94%A8spark%E8%AF%BB%E5%8F%96mysql%E6%95%B0%E6%8D%AE%E5%BA%93%E6%95%B0%E6%8D%AE%E8%BD%AC%E5%8C%96%E4%B8%BAdataframe/)