参见[Hadoop Hive sql语法详解](https://blog.csdn.net/hguisu/article/details/7256833)

- 显示所有表：SHOW TABLES;
- show functions;显示所有函数
- describe function substr;查看函数用法
- desc tbname;查看表结构
- select * from tbName;查询表数据不会做mapreduce操作
- select a.id from tbname a ;查询一列数据，会做mapreduce操作
- show partitions tbname;查看分区语句

# 从SQL到HiveQL应转变的习惯
1. Hive不支持等值连接  

```sql
-- SQL中对两表内联可以写成：
select * from dual a,dual b where a.key = b.key;
-- Hive中应为
select * from dual a join dual b on a.key = b.key; 
```

2. 分号字符
分号是SQL语句结束标记，在HiveQL中也是，但是在HiveQL中，对分号的识别没有那么智能。

```
-- 这个会报错
select concat(key,concat(';',key)) from dual;

-- 解决办法是使用分号的八进制的ASCII码进行转义
select concat(key,concat('\073',key)) from dual;
```

4. IS [NOT] NULL
SQL中null代表空值, 值得警惕的是, **在HiveQL中String类型的字段若是空(empty)字符串, 即长度为0, 那么对它进行IS NULL的判断结果是False.**