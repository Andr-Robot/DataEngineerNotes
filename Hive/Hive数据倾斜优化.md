* [导致数据倾斜的操作](#导致数据倾斜的操作)
* [调节参数](#调节参数)
* [SQL语句调节](#sql语句调节)
* [总结](#总结)
    * [使用COUNT DISTINCT和GROUP BY造成的数据倾斜](#使用count-distinct和group-by造成的数据倾斜)
    * [使用JOIN引起的数据倾斜](#使用join引起的数据倾斜)
* [参考文献](#参考文献)


# 导致数据倾斜的操作
GROUP BY, COUNT DISTINCT, join

# 调节参数
- set hive.map.aggr=true；
- set hive.groupby.skewindata=true;

> **hive.map.aggr = true**：   
在map中会做部分聚集操作，效率更高但需要更多的内存。   
**hive.groupby.skewindata = true**：   
数据倾斜时负载均衡，当选项设定为true，生成的查询计划会有**两个MRJob**。第一个MRJob 中，Map的输出结果集合会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是相同的GroupBy Key有可能被分发到不同的Reduce中，从而达到负载均衡的目的；第二个MRJob再根据预处理的数据结果按照GroupBy Key分布到Reduce中（这个过程可以保证相同的GroupBy Key被分布到同一个Reduce中），最后完成最终的聚合操作。

# SQL语句调节
1. **如何Join**：关于驱动表的选取，选用join key分布最均匀的表作为驱动表。做好列裁剪和filter操作，以达到两表做join的时候，数据量相对变小的效果。
2. **大小表Join**：对于join，在判断小表不大于1G的情况下，使用map join
3. **大表Join大表**：把空值的key变成一个字符串加上随机数，把倾斜的数据分到不同的reduce上，由于null值关联不上，处理后并不影响最终结果。
4. 采用sum() group by的方式来替换count(distinct)完成计算。

# 总结
这里列出一些常用的数据倾斜**解决办法**：
## 使用COUNT DISTINCT和GROUP BY造成的数据倾斜   
1. 存在大量空值或NULL，或者某一个值的记录特别多，可以先把该值过滤掉，在最后单独处理:   

    ```sql
    SELECT CAST(COUNT(DISTINCT imei)+1 AS bigint)
    FROM lxw1234 where pt = ‘2012-05-28′
    AND imei <> ‘lxw1234′ ;
    ```
    
    比如某一天的IMEI值为’lxw1234’的特别多，当我要统计总的IMEI数，可以先统计不为’lxw1234’的，之后再加1.

2. 多重COUNT DISTINCT   
通常使用UNION ALL + ROW_NUMBER() + SUM + GROUP BY来变通实现。

## 使用JOIN引起的数据倾斜
1. 关联键存在大量空值或者某一特殊值，如”NULL”   
    - 空值单独处理，不参与关联；   
    - 空值或特殊值加随机数作为关联键；
2. 不同数据类型的字段关联   
转换为同一数据类型之后再做关联

# 参考文献
[hadoop数据倾斜总结](https://blog.csdn.net/xinzhi8/article/details/71455883)    