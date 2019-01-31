* [对数据库操作](#对数据库操作)
* [对表操作](#对表操作)
* [约束(constraint)详解](#约束constraint详解)
* [内连接、外连接和半连接](#内连接外连接和半连接)
    * [内连接](#内连接)
    * [外连接](#外连接)
    * [半连接](#半连接)
* [Limit子句](#limit子句)
* [总结](#总结)
    * [mysql中insert into和replace into以及insert ignore用法区别](#mysql中insert-into和replace-into以及insert-ignore用法区别)

参见[Interview-Notebook——SQL](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/SQL.md)

# 对数据库操作
- **查看当前数据库服务器上的数据库**：show databases
- **创建数据库**：create database + 数据库名
- **删除数据库**：drop database + 数据库名
- **使用数据库**：use + 数据库名
- **查看数据库中所有表**：show tables

# 对表操作
- **创建表**：
    ```
    create table 表名 (
    字段名称 字段类型 字段特征(是否为null,默认值,标识列,主键,唯一键,外键,check约束),
    字段名称 字段类型 字段特征(是否为null,默认值,标识列,主键,唯一键,外键,check约束)
    )
    ```
    - 主键约束：primary key
    - 外键约束：foreign key
    - 自增：auto_increment
    - 默认值约束：default
    - 唯一键约束：unique
    - check约束：check（用于限制列中的值的范围，**MySQL不支持**）
- **删除表**：drop table + 表名
- **查看表结构**：desc + 表名
- **查询**：
    ```
    select * from 表名 where ……
    ```
    - select语句中**子句的顺序**：select,from,where,group by,having,order by,limit
    - group by：除了聚集函数，select中的每列必须出现在group by子句中
    - order by：排序，**asc：升序**；**desc：降序**，默认按升序
- **插入**：insert into 表名(‘字段名称’，‘字段名称’) values(value1,value2)
- **删除**：delete from 表名 where ……
- **修改**：update 表名 set 字段1=值，字段2=值 …… where ……
- **删除表字段**：alter table 表名  drop 字段 （注：如果数据表中**只剩余一个字段**则无法使用DROP来删除字段）
- **添加表字段**：alter table 表名 add 字段 字段类型
- **修改表字段**：alter table 表名 change 修改的字段名 修改后的字段名 修改后的字段类型
- **模糊查询**：like 子句（‘%’:匹配任何字符出现的任意次数，‘_’ : 匹配单个字符）
- **聚集函数**：avg()：平均；count()：行数；max()：最大；min()：最小；sum()：求和


# 约束(constraint)详解
参见[MySQL——约束(constraint)详解](https://blog.csdn.net/w_linux/article/details/79655073)
- 非空约束(not null)
- 唯一性约束(unique)
- 主键约束(primary key) PK
- 外键约束(foreign key) FK
- 检查约束(目前MySQL不支持、Oracle支持)

```sql
-- 创建
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT pk_PersonID PRIMARY KEY (Id_P,LastName)
)
ALTER TABLE Persons ADD PRIMARY KEY (Id_P)
ALTER TABLE Persons ADD CONSTRAINT pk_PersonID PRIMARY KEY (Id_P,LastName)

-- 删除
ALTER TABLE Persons DROP PRIMARY KEY
```

# 内连接、外连接和半连接
参见：
- [深入理解SQL的四种连接-左外连接、右外连接、内连接、全连接](https://www.jb51.net/article/39432.htm)
- [MySQL中的各种join](http://wxb.github.io/2016/12/15/MySQL%E4%B8%AD%E7%9A%84%E5%90%84%E7%A7%8Djoin.html)
- [Mysql 连接的使用](http://www.runoob.com/mysql/mysql-join.html)
- [【SQL】表连接 --半连接](https://yq.aliyun.com/articles/29839)

## 内连接
显示两个表中有联系的所有数据;
- **隐式内连接**：`select * from a,b where a.id=b.id`
- **显示内连接**：`select * from a inner join b on a.id=b.id`

## 外连接
外联接可以是**左向外联接、右向外联接或完整外部联接**。
- **左链接**：以左表为参照,显示所有数据;    
`select * from a left join b on a.id=b.id`
- **右链接**：以右表为参照显示数据;    
`select * from a right join b on a.id=b.id`
- **全连接**：左右表的数据都返回（MySQL不支持）   
`select * from a full join b on a.id=b.id`

## 半连接
半连接通常使用IN  或 EXISTS 作为连接条件。

# Limit子句
参见[深入分析Mysql中limit的用法](https://www.jb51.net/article/62851.htm)    
limit子句可以被用于强制select语句**返回指定的记录数**。limit接受**一个或两个数字参数**。参数必须是一个整数常量。如果给定两个参数，第一个参数指定第一个返回记录行的偏移量，第二个参数指定返回记录行的最大数目。初始记录行的偏移量是 0(而不是 1)。

**SELECT * FROM table   LIMIT [offset,] rows | rows OFFSET offset**

```sql
select * from book limit 5,10;  /*取得记录行6-15的记录*/
select * from book limit 5,-1;  /*取得记录行6-末尾的记录*/
select * from book limit 5;  /*取得记录行前5个的记录，等价于limit 0,5*/
select * from book limit 5,1;  /*取得第5行的后1行的记录*/
```

**性能分析**：取90000条后100条记录。

```sql
-- 方法1
Select * From cyclopedia Where ID>=(
Select Max(ID) From (
Select ID From cyclopedia Order By ID limit 90001
) As tmp
) limit 100;

-- 方法2
Select * From cyclopedia Where ID>=(
Select ID From cyclopedia limit 90000,1
)limit 100;
```

第1句是先取了前90001条记录,取其中最大一个ID值作为起始标识,然后利用它可以快速定位下100条记录。    
第2句则是仅仅取90000条记录后1条,然后取ID值作起始标识定位下100条记录    
第1句执行结果.100 rows in set (0.23) sec    
第2句执行结果.100 rows in set (0.19) sec    
**明显第2种执行效率更高**。

**总结**：
- offset比较小的时候，直接使用limit较优。(SQL 1更优)

    ```sql
    -- SQL 1
    select * from yanxue8_visit limit 10,10;
    -- SQL 2
    Select * From yanxue8_visit Where vid >=(
    Select vid From yanxue8_visit Order By vid limit 10,1
    ) limit 10;
    ```

- offset大的时候，使用offset更优。(SQL 2更优)

    ```sql
    -- SQL 1
    select * from yanxue8_visit limit 10000,10;
    -- SQL 2
    Select * From yanxue8_visit Where vid >=(
    Select vid From yanxue8_visit Order By vid limit 10000,1
    ) limit 10;
    ```

# 总结
## mysql中insert into和replace into以及insert ignore用法区别
- **insert into**表示**插入数据**，**数据库会检查主键**，如果出现重复会报错； 
- **replace into**表示**插入替换数据**，需求表中有PrimaryKey，或者unique索引，**如果数据库已经存在数据，则用新数据替换**，如果没有数据效果则和insert into一样； 
- **insert ignore**表示**如果表中已经存在相同的记录（根据主键primary或者唯一索引unique判断），则忽略当前新数据**。 