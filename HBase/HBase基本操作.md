名称 | 命令表达式
---|---
查看存在哪些表 | list
创建表 | create ‘表名称’, ‘列名称1’,’列名称2’,’列名称N’
添加记录 | put ‘表名称’, ‘行名称’, ‘列名称:’, ‘值’
查看记录 | get ‘表名称’, ‘行名称’
查看表中的记录总数 | count ‘表名称’
删除记录 | delete ‘表名’ ,’行名称’ , ‘列名称’
删除一张表 | 先要屏蔽该表，才能对该表进行删除，第一步 disable ‘表名称’ 第二步 drop ‘表名称’
查看所有记录 | scan “表名称”
查看某个表某个列中所有数据 | scan “表名称” , [‘列名称:’]
更新记录 | 就是重写一遍进行覆盖

**注意**：Hbase 基于**非行健值**查询的唯一途径是通过带过滤器的扫描。（**HBase不支持事务**）

```sql
-- 创建表
create <table>, {NAME => <family>, VERSIONS => <VERSIONS>}
-- 添加数据
put <table>,<rowkey>,<family:column>,<value>,<timestamp>
-- 查询某行记录
get <table>,<rowkey>,[<family:column>,....]
-- 扫描表
scan <table>, {COLUMNS => [ <family:column>,.... ], LIMIT => num}
-- 扫描表t1的前5条数据
scan 't1',{LIMIT=>5}
--删除行中的某个列值,必须指定列名.
delete <table>, <rowkey>, <family:column> , <timestamp>
-- 删除行,可以不指定列名，删除整行数据.
deleteall <table>, <rowkey>, <family:column> , <timestamp>
-- 查询的是表名为testByCrq，过滤方式是通过value过滤，匹配出value含111的数据
scan 'testByCrq', FILTER=>"ValueFilter(=,'substring:111')"
```

**获取记录方式**：
- 通过get方式，指定rowkey获取唯一记录
- 通过scan方式，设置startRow和stopRow参数进行范围匹配
- 通过scan方式，全表扫描，并通过RowFilter过滤出数据
- 基本可归纳为顺序读和随机读。

# 参考文献
[HBase shell 命令介绍](http://www.ityouknow.com/hbase/2017/07/28/hbase-shell.html)    
[HBase Shell 常用操作](http://debugo.com/hbase-shell-cmds/)   
[HBase常用Shell命令和基础开发](https://cshihong.github.io/2018/06/11/HBase%E5%B8%B8%E7%94%A8Shell%E5%91%BD%E4%BB%A4%E5%92%8C%E5%9F%BA%E7%A1%80%E5%BC%80%E5%8F%91/)   
[HBASE SHELL常用命令](https://www.jianshu.com/p/2e0aac25aef9)    
[Hbase Shell常用命令](https://blog.liyang.io/159.html)    
[在hbase shell中过滤器的简单使用](https://blog.csdn.net/qq_27078095/article/details/56482010)