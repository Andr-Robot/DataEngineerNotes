[toc]

# 案例分析
> 使用sql实现计算90分位。有10000个用户，每个用户有一个user_id和对应的交易量统计值cnt。按照交易量从大到小排序，挑选能构成总交易量占比的前90%的用户。

蚂蚁金服面试题，这道题需要统计构成总交易量占比的前90%的用户，因为已经是排好序的数据了，接下来我们需要将每行求出从第一行到当前行的总和cursum。然后根据cursum去和总交易量做对比，便可以找出符合题意的用户信息。   
**注意：** 在这里使用serverid和num来类比原题。
```sql
select serverid,num from serverinfo order by num desc

输出：
serverid num
10118 2116
10001 1499
10119 1206
10060 1073
10031 965
10117 759
10071 691
10114 670
10077 611
10085 544
10095 532
10112 490
10103 433
10108 394
10105 338
10050 317
10116 270
10107 202
10113 201
10073 124
10115 22
```

1. 统计cursum(第一行到当前行的和)

```sql
select serverid,num,sum(num) over(order by num desc) as curnum from (select serverid,num from serverinfo order by num desc) as t

输出：
serverid num curnum
10118 2116 2116
10001 1499 3615
10119 1206 4821
10060 1073 5894
10031 965 6859
10117 759 7618
10071 691 8309
10114 670 8979
10077 611 9590
10085 544 10134
10095 532 10666
10112 490 11156
10103 433 11589
10108 394 11983
10105 338 12321
10050 317 12638
10116 270 12908
10107 202 13110
10113 201 13311
10073 124 13435
10115 22 13457
```

2. 根据全表中总的num数值乘以90%得到的值来筛选符合要求的serverid

```sql
select serverid,num,curnum from (select serverid,num,sum(num) over(order by num desc) as curnum from (select serverid,num from serverinfo order by num desc) as t) as t1 where t1.curnum <= ((select sum(num) from serverinfo) * 0.9)

输出：
serverid num curnum
10118 2116 2116
10001 1499 3615
10119 1206 4821
10060 1073 5894
10031 965 6859
10117 759 7618
10071 691 8309
10114 670 8979
10077 611 9590
10085 544 10134
10095 532 10666
10112 490 11156
10103 433 11589
10108 394 11983
```

# over函数

```sql
OVER (   
       [ <PARTITION BY clause> ]  
       [ <ORDER BY clause> ]   
       [ <ROWS BETWEEN ...> ]  
      ) 
```

- 如果不指定ROWS BETWEEN,默认为从起点到当前行;
- 如果不指定ORDER BY，则将分组内所有值累加;
- 关键是理解ROWS BETWEEN含义,也叫做WINDOW子句：
    - PRECEDING：往前
    - FOLLOWING：往后
    - CURRENT ROW：当前行
    - UNBOUNDED：起点，UNBOUNDED PRECEDING 表示从前面的起点， UNBOUNDED FOLLOWING：表示到后面的终点

# SUM,AVG,MIN,MAX
用于实现分组内所有和连续累积的统计。
参见[Hive分析窗口函数(一) SUM,AVG,MIN,MAX](http://lxw1234.com/archives/2015/04/176.htm)

原数据：
```sql
select * from lxw1234;
OK
cookie1 2015-04-10      1
cookie1 2015-04-11      5
cookie1 2015-04-12      7
cookie1 2015-04-13      3
cookie1 2015-04-14      2
cookie1 2015-04-15      4
cookie1 2015-04-16      4
```

SUM操作：

```sql
SELECT cookieid,
createtime,
pv,
SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime) AS pv1, -- 默认为从起点到当前行
SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS pv2, --从起点到当前行，结果同pv1 
SUM(pv) OVER(PARTITION BY cookieid) AS pv3,								--分组内所有行
SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN 3 PRECEDING AND CURRENT ROW) AS pv4,   --当前行+往前3行
SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN 3 PRECEDING AND 1 FOLLOWING) AS pv5,    --当前行+往前3行+往后1行
SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) AS pv6   ---当前行+往后所有行  
FROM lxw1234;
 
cookieid createtime     pv      pv1     pv2     pv3     pv4     pv5      pv6 
-----------------------------------------------------------------------------
cookie1  2015-04-10      1       1       1       26      1       6       26
cookie1  2015-04-11      5       6       6       26      6       13      25
cookie1  2015-04-12      7       13      13      26      13      16      20
cookie1  2015-04-13      3       16      16      26      16      18      13
cookie1  2015-04-14      2       18      18      26      17      21      10
cookie1  2015-04-15      4       22      22      26      16      20      8
cookie1  2015-04-16      4       26      26      26      13      13      4
```

AVG，MIN，MAX，和SUM用法一样

# NTILE,ROW_NUMBER,RANK,DENSE_RANK
这一类主要用于**排序编号**，分组排序编号，取前topn，分组百分比和组内比例等用途。   
**注意： 这类序列函数不支持WINDOW子句。**   
参见[Hive分析窗口函数(二) NTILE,ROW_NUMBER,RANK,DENSE_RANK](http://lxw1234.com/archives/2015/04/181.htm)。   
原数据：

```sql
select * from lxw1234;
OK
cookie1 2015-04-10      1
cookie1 2015-04-11      5
cookie1 2015-04-12      7
cookie1 2015-04-13      3
cookie1 2015-04-14      2
cookie1 2015-04-15      4
cookie1 2015-04-16      4
cookie2 2015-04-10      2
cookie2 2015-04-11      3
cookie2 2015-04-12      5
cookie2 2015-04-13      6
cookie2 2015-04-14      3
cookie2 2015-04-15      9
cookie2 2015-04-16      7
```

## NTILE
NTILE(n)，用于将分组数据按照顺序切分成n片，返回当前切片值。   
> 统计一个cookie，pv数最多的前1/3的天。

```sql
SELECT 
cookieid,
createtime,
pv,
NTILE(3) OVER(PARTITION BY cookieid ORDER BY pv DESC) AS rn 
FROM lxw1234;
 
--rn = 1 的记录，就是我们想要的结果
 
cookieid day           pv       rn
----------------------------------
cookie1 2015-04-12      7       1
cookie1 2015-04-11      5       1
cookie1 2015-04-15      4       1
cookie1 2015-04-16      4       2
cookie1 2015-04-13      3       2
cookie1 2015-04-14      2       3
cookie1 2015-04-10      1       3
cookie2 2015-04-15      9       1
cookie2 2015-04-16      7       1
cookie2 2015-04-13      6       1
cookie2 2015-04-12      5       2
cookie2 2015-04-14      3       2
cookie2 2015-04-11      3       3
cookie2 2015-04-10      2       3
```

## ROW_NUMBER
ROW_NUMBER()，从1开始，按照顺序，生成分组内记录的序列。
> 按照pv降序排列，生成分组内每天的pv名次。

```sql
SELECT 
cookieid,
createtime,
pv,
ROW_NUMBER() OVER(PARTITION BY cookieid ORDER BY pv desc) AS rn 
FROM lxw1234;
 
cookieid day           pv       rn
------------------------------------------- 
cookie1 2015-04-12      7       1
cookie1 2015-04-11      5       2
cookie1 2015-04-15      4       3
cookie1 2015-04-16      4       4
cookie1 2015-04-13      3       5
cookie1 2015-04-14      2       6
cookie1 2015-04-10      1       7
cookie2 2015-04-15      9       1
cookie2 2015-04-16      7       2
cookie2 2015-04-13      6       3
cookie2 2015-04-12      5       4
cookie2 2015-04-14      3       5
cookie2 2015-04-11      3       6
cookie2 2015-04-10      2       7
```

## RANK 和 DENSE_RANK
- RANK() 生成数据项在分组中的排名，排名相等会在名次中留下空位
- DENSE_RANK() 生成数据项在分组中的排名，排名相等会在名次中不会留下空位

```sql
SELECT 
cookieid,
createtime,
pv,
RANK() OVER(PARTITION BY cookieid ORDER BY pv desc) AS rn1,
DENSE_RANK() OVER(PARTITION BY cookieid ORDER BY pv desc) AS rn2,
ROW_NUMBER() OVER(PARTITION BY cookieid ORDER BY pv DESC) AS rn3 
FROM lxw1234 
WHERE cookieid = 'cookie1';
 
cookieid day           pv       rn1     rn2     rn3 
-------------------------------------------------- 
cookie1 2015-04-12      7       1       1       1
cookie1 2015-04-11      5       2       2       2
cookie1 2015-04-15      4       3       3       3
cookie1 2015-04-16      4       3       3       4
cookie1 2015-04-13      3       5       4       5
cookie1 2015-04-14      2       6       5       6
cookie1 2015-04-10      1       7       6       7
 
rn1: 15号和16号并列第3, 13号排第5
rn2: 15号和16号并列第3, 13号排第4
rn3: 如果相等，则按记录值排序，生成唯一的次序，如果所有记录值都相等，或许会随机排吧。
```

# CUME_DIST,PERCENT_RANK
- CUME_DIST()：小于等于当前值的行数/分组内总行数
- PERCENT_RANK()：分组内当前行的RANK值-1/分组内总行数-1

参见[Hive分析窗口函数(三) CUME_DIST,PERCENT_RANK](http://lxw1234.com/archives/2015/04/185.htm)

# LAG,LEAD,FIRST_VALUE,LAST_VALUE
- LAG(col,n,DEFAULT)：用于统计窗口内往上第n行值
第一个参数为列名，第二个参数为往上第n行（可选，默认为1），第三个参数为默认值（当往上第n行为NULL时候，取默认值，如不指定，则为NULL）
- LEAD(col,n,DEFAULT)：用于统计窗口内往下第n行值
第一个参数为列名，第二个参数为往下第n行（可选，默认为1），第三个参数为默认值（当往下第n行为NULL时候，取默认值，如不指定，则为NULL）
- FIRST_VALUE()：取分组内排序后，截止到当前行，第一个值
- LAST_VALUE()：取分组内排序后，截止到当前行，最后一个值

参见[Hive分析窗口函数(四) LAG,LEAD,FIRST_VALUE,LAST_VALUE](http://lxw1234.com/archives/2015/04/190.htm)

# GROUPING SETS,GROUPING__ID,CUBE,ROLLUP
- GROUPING SETS：在一个GROUP BY查询中，根据不同的维度组合进行聚合，等价于将不同维度的GROUP BY结果集进行UNION ALL
- CUBE：根据GROUP BY的维度的所有组合进行聚合
- ROLLUP：是CUBE的子集，以最左侧的维度为主，从该维度进行层级聚合

参见[Hive分析窗口函数(五) GROUPING SETS,GROUPING__ID,CUBE,ROLLUP](http://lxw1234.com/archives/2015/04/193.htm)

# 参考文献
[hive分析函数](http://lxw1234.com/archives/tag/hive-window-functions)   
[SELECT - OVER 子句 (Transact-SQL)](https://docs.microsoft.com/zh-cn/sql/t-sql/queries/select-over-clause-transact-sql?view=sql-server-2017)   

