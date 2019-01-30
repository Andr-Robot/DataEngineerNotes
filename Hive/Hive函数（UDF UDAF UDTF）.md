* [UDF](#udf)
* [UDAF](#udaf)
* [UDTF](#udtf)
* [部署函数](#部署函数)
* [Hive常用的函数介绍](#hive常用的函数介绍)
    * [collect_set和collect_list](#collect_set和collect_list)
    * [concat和concat_ws](#concat和concat_ws)
    * [get_json_object](#get_json_object)
    * [explode](#explode)
* [参考文献](#参考文献)


# UDF
普通的用户自定义函数。**接受单行输入，并产生单行输出**。继承UDF，重写evaluate方法即可。

```java
package com.example.hive.udf;
 
import org.apache.hadoop.hive.ql.exec.UDF;
import org.apache.hadoop.io.Text;

public final class Lower extends UDF {
  public Text evaluate(final Text s) {
    if (s == null) { return null; }
    return new Text(s.toString().toLowerCase());
  }
}
```


# UDAF
用户定义聚集函数（User-defined aggregate function）。**接受多行输入，并产生单行输出**。比如MAX，COUNT函数。

步骤：
1. 必须继承
    - org.apache.hadoop.hive.ql.exec.UDAF(函数类继承)
    - org.apache.hadoop.hive.ql.exec.UDAFEvaluator(内部类Evaluator实现UDAFEvaluator接口)
2. Evaluator需要实现 init、iterate、terminatePartial、merge、terminate这几个函数
    - init():类似于构造函数，用于UDAF的初始化
    - iterate():接收传入的参数，并进行内部的轮转。其返回类型为boolean
    - terminatePartial():无参数，其为iterate函数轮转结束后，返回乱转数据，iterate和terminatePartial类似于hadoop的Combiner(iterate--mapper;terminatePartial--reducer)
    - merge():接收terminatePartial的返回结果，进行数据merge操作，其返回类型为boolean
    - terminate():返回最终的聚集函数结果

实例参见[Hive中UDF、UDAF和UDTF使用](https://blog.csdn.net/fover717/article/details/64926854) 

# UDTF
用户定义表生成函数（User-defined table-generating function）。**接受单行输入，并产生多行输出（即一个表）**。

步骤：
1. 必须继承org.apache.hadoop.hive.ql.udf.generic.GenericUDTF
2. 实现initialize, process, close三个方法
3. UDTF首先会
    - 调用initialize方法，此方法返回UDTF的返回行的信息（返回个数，类型）
    - 初始化完成后，会调用process方法，对传入的参数进行处理，可以通过forword()方法把结果返回
    - 最后close()方法调用，对需要清理的方法进行清理

实例参见[Hive中UDF、UDAF和UDTF使用](https://blog.csdn.net/fover717/article/details/64926854) 

# 部署函数

```
这个是最常见的Hive使用方式，通过hive的命令来完成UDF的部署
hive> add jar xxxx.jar
hive> CREATE TEMPORARY FUNCTION function_name AS 'udf类路径'; 这种方法的的缺点很明显就是每次都需要使用这两个命令来完成操作
```

具体参见[hive 部署UDF函数](https://yq.aliyun.com/articles/499562)

# Hive常用的函数介绍
## collect_set和collect_list
- collect_set(col)函数只接受基本数据类型，它的主要作用是将某字段的值进行去重汇总，产生array类型字段。
- collect_list(col)函数只接受基本数据类型，它的主要作用是将某字段的值进行汇总，产生array类型字段。

## concat和concat_ws

 返回类型 | 函数名 | 描述
---|---|---
string | concat(string\|binary A, string\|binary B...) | 返回将A和B按顺序连接在一起的字符串，如：concat('foo', 'bar') 返回'foobar'
string | concat_ws(string SEP, string A, string B...) | 类似concat() ，但连接时会使用自定义的分隔符SEP
string | concat_ws(string SEP, array<string>) | 类似concat_ws() ，但参数为字符串数组

## get_json_object
json解析函数。
- 语法: `get_json_object(string json_string, string path)`
- 返回值: `string`
- 说明：解析json的字符串json_string,返回path指定的内容。如果输入的json字符串无效，那么返回NULL。


```sql
hive> select get_json_object('{"store":
>  {"fruit":\[{"weight":8,"type":"apple"},{"weight":9,"type":"pear"}],
>   "bicycle":{"price":19.95,"color":"red"}
>   },
> "email":"amy@only_for_json_udf_test.net",
>  "owner":"amy"
> }
> ','$.owner') from iteblog;
amy
```

## explode
explode(array|map)函数接受array或map类型的参数，其作用恰好与collect_set相反，实现将array或map类型数据行转列。
详情参见[Hive-explode[列转行]关键字使用](https://my.oschina.net/zhzhenqin/blog/602536)和[Lateral View用法 与 Hive UDTF explode](https://blog.csdn.net/oopsoom/article/details/26001307)    

**注意**：在select语句中使用UDTF语法的限制：
- 在select语句中只支持单个UDTF表达式
- 不支持UDTF嵌套
- 不支持GROUP BY / CLUSTER BY / DISTRIBUTE BY / SORT BY语句

所以需要使用**Lateral View**。Lateral view 其实就是用来和像类似explode这种UDTF函数联用的。**lateral view 会将UDTF生成的结果放到一个虚拟表中，然后这个虚拟表会和输入行每一行进行join 来达到连接UDTF外的select字段的目的**。

**注意：** 在Hive0.12里面会支持**outer**关键字，如果UDTF的结果是空，默认会被忽略输出。
如果加上outer关键字，则会像left outer join 一样，还是会输出select出的列，而UDTF的输出结果是NULL。

```
hive> select * from explode_map;
OK
00001	zhzhenqin	{"80":1,"90":2}
00002	hello	{"java":10,"女":2}
00003	world	{"java":1,"python":3,"90":1}
Time taken: 0.04 seconds, Fetched: 3 row(s)

select userId,userName,tagId,weight from explode_map 
lateral view explode(tags) tags as tagId, weight;
00001	zhzhenqin	80	1
00001	zhzhenqin	90	2
00002	hello	java	10
00002	hello	女	2
00003	world	java	1
00003	world	python	3
00003	world	90	1

SELECT myCol1, myCol2 FROM baseTable
LATERAL VIEW explode(col1) myTable1 AS myCol1
LATERAL VIEW explode(col2) myTable2 AS myCol2;

-- Outer Lateral View
select * FROM test_lateral_view_shengli LATERAL VIEW explode(array()) C AS a ;
SELECT * FROM src LATERAL VIEW OUTER explode(array()) C AS a limit 10;
```



# 参考文献
[Hive自定义函数（UDF、UDAF、UDTF）与SerDe](http://www.zhangrenhua.com/2016/02/21/hadoop-Hive%E8%87%AA%E5%AE%9A%E4%B9%89UDF%E4%B8%8ESerDe/)   
[Hive 10、Hive的UDF、UDAF、UDTF](https://www.cnblogs.com/raphael5200/p/5215337.html)   
[Hive中UDF、UDAF和UDTF使用](https://blog.csdn.net/fover717/article/details/64926854)   
[hive 部署UDF函数](https://yq.aliyun.com/articles/499562)    
[Hive常用字符串函数](https://www.iteblog.com/archives/1639.html)   
[Hive的Collect函数](https://blog.csdn.net/u014307117/article/details/52296757)   
[Hive-explode[列转行]关键字使用](https://my.oschina.net/zhzhenqin/blog/602536)   
[Hive学习之Lateral View](https://blog.csdn.net/skywalker_only/article/details/39289709)   
[Lateral View用法 与 Hive UDTF explode](https://blog.csdn.net/oopsoom/article/details/26001307)    
[Hive UDF函数开发使用样例](https://sjq597.github.io/2015/11/25/Hive-UDF%E5%87%BD%E6%95%B0%E5%BC%80%E5%8F%91%E4%BD%BF%E7%94%A8%E6%A0%B7%E4%BE%8B/)