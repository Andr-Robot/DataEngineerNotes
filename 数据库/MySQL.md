* [存储引擎](#存储引擎)
    * [数据库引擎](#数据库引擎)
    * [InnoDB](#innodb)
    * [MyISAM](#myisam)
    * [比较](#比较)
    * [选择](#选择)
* [索引](#索引)
    * [索引实现类型](#索引实现类型)
    * [MySQL中MyIASM和Innodb的B+ 树索引](#mysql中myiasm和innodb的b-树索引)
    * [MySQL索引分类](#mysql索引分类)
    * [索引使用原则](#索引使用原则)
        * [列独立](#列独立)
        * [左前缀](#左前缀)
        * [复合索引由左到右生效](#复合索引由左到右生效)
        * [不要滥用索引，多余的索引会降低读写性能](#不要滥用索引多余的索引会降低读写性能)
    * [索引选择原则](#索引选择原则)
    * [索引的优缺点](#索引的优缺点)
* [MySQL复制](#mysql复制)
    * [MySQL复制的原理](#mysql复制的原理)
* [MySQL读写分离](#mysql读写分离)
* [MySQL中SQL执行顺序](#mysql中sql执行顺序)
* [总结](#总结)
    * [OLTP(联机事务处理)系统和OLAP(联机分析处理)系统的区别](#oltp联机事务处理系统和olap联机分析处理系统的区别)
* [参考文献](#参考文献)

参见[Interview-Notebook——MySQL](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/MySQL.md)

# 存储引擎
## 数据库引擎
当你访问数据库时，不管是手工访问，还是程序访问，都不是直接读写数据库文件，而是**通过数据库引擎去访问数据库文件**。以关系型数据库为例，你发SQL语句给数据库引擎，数据库引擎解释SQL语句，提取出你需要的数据返回给你。因此，对访问者来说，**数据库引擎就是SQL语句的解释器**。数据库引擎是用于**存储、处理和保护数据**的核心服务。利用数据库引擎可以控制访问权限并快速处理事务。

## InnoDB
**事务型存储引擎**，是 MySQL 默认的，只有在需要它不支持的特性时，才考虑使用其它存储引擎。

**实现了四个标准的隔离级别，默认级别是可重复读**（REPEATABLE READ）。在可重复读隔离级别下，通过多版本并发控制（MVCC）+ 间隙锁（Next-Key Locking）防止幻影读。

**主索引（主码索引）是聚簇索引，在索引中保存了数据**，从而避免直接读取磁盘，因此对查询性能有很大的提升。

MySQL运行时Innodb会**在内存中建立缓冲池**，用于缓冲数据和索引。但是该引擎**不支持FULLTEXT类型的索引**，而且它**没有保存表的行数**，当`SELECT COUNT(*) FROM TABLE`时需要扫描全表。

**适用范围**：如果需要事务支持，并且有较高的并发读取频率，InnoDB是不错的选择。

## MyISAM
它**没有提供对数据库事务的支持，也不支持行级锁和外键**，因此当INSERT(插入)或UPDATE(更新)数据时即写操作需要**锁定整个表**，效率便会低一些。

MyISAM中**存储了表的行数**，于是`SELECT COUNT(*) FROM TABLE`时只需要直接读取已经保存好的值而不需要进行全表扫描。

每当我们建立一个MyISAM引擎的表时，就会在本地磁盘上建立三个文件，文件名就是表名。例如，我建立了一个MyISAM引擎的tb_Demo表，那么就会生成以下三个文件：
- tb_demo.frm，存储表定义。
- tb_demo.MYD，存储数据。
- tb_demo.MYI，存储索引。

**适用范围**：如果表的读操作远远多于写操作且不需要数据库事务的支持，那么MyIASM也是很好的选择。

## 比较
- 事务：InnoDB 是事务型的，可以使用 Commit 和 Rollback 语句。
- 并发：MyISAM 只支持表级锁，而 InnoDB 还支持行级锁。
- 外键：InnoDB 支持外键，MyISAM不支持。
- 备份：InnoDB 支持在线热备份。
- 崩溃恢复：MyISAM 崩溃后发生损坏的概率比 InnoDB 高很多，而且恢复的速度也更慢。
- 其它特性：MyISAM 支持压缩表和空间数据索引。

## 选择
- 大尺寸的数据集趋向于选择InnoDB引擎，因为它支持事务处理和故障恢复。
- 主键查询在InnoDB引擎下也会相当快，不过需要注意的是如果主键太长也会导致性能问题。
- 大批的INSERT语句在MyISAM下会快一些，但是UPDATE语句在InnoDB下则会更快一些，尤其是在并发量大的时候。

# 索引
索引是在**存储引擎层**实现的，而不是在服务器层实现的，所以不同存储引擎具有不同的索引类型和实现。

## 索引实现类型
- B+Tree 索引
- 哈希索引：哈希索引能以 O(1) 时间进行查找，但是失去了有序性，它具有以下限制：
    - 无法用于排序与分组
    - 只支持精确查找，无法用于部分查找和范围查找
- 全文索引：用于**查找文本中的关键词**，而不是直接比较是否相等。查找条件使用 **MATCH AGAINST**，而不是普通的 WHERE。全文索引一般使用**倒排索引**实现，**它记录着关键词到其所在文档的映射**。
- 空间数据索引：MyISAM 存储引擎支持**空间数据索引**（R-Tree），可以**用于地理数据存储**。空间数据索引会从所有维度来索引数据，可以有效地使用任意维度来进行组合查询。必须使用 GIS 相关的函数来维护数据。

## MySQL中MyIASM和Innodb的B+ 树索引
索引（Index）是帮助MySQL高效获取数据的数据结构。**MyIASM和Innodb都使用了树这种数据结构做为索引**。

- MyISAM引擎的索引结构为**B+Tree**，其中B+Tree的**数据域存储的内容为实际数据的地址**，也就是说它的索引和实际的数据是分开的，只不过是用索引指向了实际的数据，这种索引就是所谓的**非聚集索引**。
- InnoDB引擎的索引结构同样也是**B+Tree**，InnoDB 的 B+Tree 索引分为**主索引**和**辅助索引**。
    - 主索引的叶子节点 data 域记录着完整的数据记录，这种索引方式被称为聚簇索引。因为无法把数据行存放在两个不同的地方，所以一个表只能有一个**聚簇索引**。
    - 辅助索引的叶子节点的 data 域记录着主键的值，因此在使用辅助索引进行查找时，需要先查找到主键值，然后再到主索引中进行查找。

**注意：**
- 因为InnoDB的数据文件本身要按主键聚集，所以**InnoDB要求表必须有主键**（MyISAM可以没有），如果没有显式指定，则MySQL系统会自动选择一个可以唯一标识数据记录的列作为主键，如果不存在这种列，则MySQL自动为InnoDB表生成一个隐含字段作为主键，这个字段长度为6个字节，类型为长整形。
- 并且和MyISAM不同，**InnoDB的辅助索引数据域存储的也是相应记录主键的值而不是地址**，所以当以辅助索引查找时，会先根据辅助索引找到主键，再根据主键索引找到实际的数据。所以**Innodb不建议使用过长的主键**，否则会使辅助索引变得过大。建议使用自增的字段作为主键，这样B+Tree的每一个结点都会被顺序的填满，而不会频繁的分裂调整，会有效的提升插入数据的效率。

## MySQL索引分类
- **普通索引**：这是最基本的索引，它没有任何限制。
    
    ```sql
    -- 创建索引
    CREATE INDEX indexName ON mytable(username(length)); 
    -- 修改表结构(添加索引)
    ALTER table tableName ADD INDEX indexName(columnName);
    -- 创建表的时候直接指定
    CREATE TABLE mytable(
    ID INT NOT NULL,   
    username VARCHAR(16) NOT NULL,  
    INDEX [indexName] (username(length))  
    );  
    -- 删除索引的语法
    DROP INDEX [indexName] ON mytable; 
    ```

- **唯一索引**：它与前面的普通索引类似，不同的就是：**索引列的值必须唯一**，但允许有空值。如果是组合索引，则列值的组合必须唯一。

    ```sql
    -- 创建索引
    CREATE UNIQUE INDEX indexName ON mytable(username(length)) 
    -- 修改表结构
    ALTER table mytable ADD UNIQUE [indexName] (username(length))
    -- 创建表的时候直接指定
    CREATE TABLE mytable(  
    ID INT NOT NULL,   
    username VARCHAR(16) NOT NULL,  
    UNIQUE [indexName] (username(length))  
    );  
    -- 使用ALTER 命令添加和删除索引，有四种方式来添加数据表的索引：
    -- 该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL。
    ALTER TABLE tbl_name ADD PRIMARY KEY (column_list)
    -- 这条语句创建索引的值必须是唯一的（除了NULL外，NULL可能会出现多次）。
    ALTER TABLE tbl_name ADD UNIQUE index_name (column_list)
    -- 添加普通索引，索引值可出现多次。
    ALTER TABLE tbl_name ADD INDEX index_name (column_list)
    -- 该语句指定了索引为 FULLTEXT ，用于全文索引。
    ALTER TABLE tbl_name ADD FULLTEXT index_name (column_list)
    -- 使用 ALTER 命令删除主键，删除主键时只需指定PRIMARY KEY，但在删除索引时，你必须知道索引名。
    ALTER TABLE testalter_tbl DROP PRIMARY KEY
    -- 在 ALTER 命令中使用 DROP 子句来删除非主键索引
    ALTER TABLE testalter_tbl DROP INDEX c
    ```

## 索引使用原则
### 列独立
保证索引包含的字段独立在查询语句中，不能是在表达式中（索引列不能参与计算）。

### 左前缀
索引可以简单如一个列(a)，也可以复杂如多个列(a, b, c, d)，即联合索引。如果是**联合索引**，那么key也由多个列组成，同时，索引只能用于查找key是否存在（相等），**遇到范围查询(>、<、between、like左匹配)等就不能进一步匹配了，后续退化为线性查找。** 因此，列的排列顺序决定了可命中索引的列数。    

如有索引`(a, b, c, d)`，查询条件`a = 1 and b = 2 and c > 3 and d = 4`，则会在每个节点依次命中a、b、c，无法命中d。也就是最左前缀匹配原则。

**=、in自动优化顺序**:   
不需要考虑=、in等的顺序，mysql会自动优化这些条件的顺序，以匹配尽可能多的索引列。    

如有索引(a, b, c, d)，查询条件`c > 3 and b = 2 and a = 1 and d < 4`与`a = 1 and c > 3 and b = 2 and d < 4`等顺序都是可以的，MySQL会自动优化为`a = 1 and b = 2 and c > 3 and d < 4`，依次命中a、b、c。

### 复合索引由左到右生效
建立联合索引，要同时考虑列查询的频率和列的区分度。   
如：index(a,b,c)

语句 | 索引是否发挥作用
---|---
where a=3 | 是，只使用了a
where a=3 and b=5 | 是，使用了a,b
where a=3 and b=5 and c=4 | 是，使用了a,b,c
where b=3 or where c=4 | 否
where a=3 and c=4 | 是，仅使用了a
where a=3 and b>10 and c=7 | 是，使用了a,b
where a=3 and b like '%xx%' and c=7 | 使用了a,b

**注意：** or的两边都有存在可用的索引，该语句才能用索引。


### 不要滥用索引，多余的索引会降低读写性能

## 索引选择原则
1. 较频繁的作为查询条件的字段应该创建索引
2. 唯一性太差的字段不适合单独创建索引，即使频繁作为查询条件
3. 更新非常频繁的字段不适合创建索引
4. 不会出现在 WHERE 子句中的字段不该创建索引

## 索引的优缺点
**索引的优点**：
1. 通过创建唯一性索引，可以保证数据库表中每一行数据的唯一性。
2. 可以大大加快数据的检索速度，这也是创建索引的最主要的原因。
3. 可以加速表和表之间的连接，特别是在实现数据的参考完整性方面特别有意义。
4. 在使用分组和排序子句进行数据检索时，同样可以显著减少查询中分组和排序的时间。
5. 通过使用索引，可以在查询的过程中，使用优化隐藏器，提高系统的性能。

**索引的缺点**：
1. 创建索引和维护索引要耗费时间，这种时间随着数据量的增加而增加。 
2. 索引需要占物理空间，除了数据表占数据空间之外，每一个索引还要占一定的物理空间，如果要建立聚簇索引，那么需要的空间就会更大。
3. 当对表中的数据进行增加、删除和修改的时候，索引也要动态的维护，这样就降低了数据的维护速度。

# MySQL复制
## MySQL复制的原理
**将数据分布到多个系统上去，是通过将MySQL的某一台master主机的数据复制到其它（slave）主机上，并重新执行一遍来实现的**。

复制过程中一个服务器充当master服务器，而一台或多台其它服务器充当slave服务器。master服务器将更新写入二进制日志文件，并维护文件的一个索引以跟踪日志循环。这些日志可以记录发送到slave服务器的更新。当一个slaves服务器连接master服务器时，它通知master服务器从服务器在日志中读取的最后一次成功更新的位置。slave服务器接收从那时起发生的任何更新，然后封锁并等待master服务器通知新的更新。将master服务器中主数据库的ddl和dml操作通过二进制日志传到slaves服务器上，然后在master服务器上将这些日志文件重新执行，从而使得slave服务器和master服务器上的数据信息保持同步。
- MySQL**主从模式**是对主操作数据，从会实时同步数据。反之对从操作，主不会同步数据，还有可能造成数据紊乱，导致主从失效。
- MySQL**主主模式**是互为对方的从服务器，每台服务器即是对方的主服务器，又是对方的从服务器。无论对哪一台进行操作，另一台都会同步数据。一般用作高容灾方案。

主要涉及三个线程：**binlog 线程、I/O 线程和 SQL 线程**。
- **binlog 线程** ：负责将主服务器上的数据更改写入二进制日志中。
- **I/O 线程** ：负责从主服务器上读取二进制日志，并写入从服务器的中继日志中。
- **SQL 线程** ：负责读取中继日志并重放其中的 SQL 语句。

![master-slave](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/mysqlmaster-slave.png)

# MySQL读写分离
主服务器处理写操作以及实时性要求比较高的读操作，而从服务器处理读操作。

读写分离能提高性能的原因在于：
- 主从服务器负责各自的读和写，极大程度缓解了锁的争用；
- 从服务器可以使用 MyISAM，提升查询性能以及节约系统开销；
- 增加冗余，提高可用性。

读写分离常用代理方式来实现，代理服务器接收应用层传来的读写请求，然后决定转发到哪个服务器。
![master-slave-proxy](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/mysqlmaster-slave-proxy.png)

# MySQL中SQL执行顺序
MySQL的语句一共分为11步，如下图所标注的那样，最先执行的总是FROM操作，最后执行的是LIMIT操作。其中每一个操作都会产生一张虚拟的表，这个虚拟的表作为一个处理的输入，只是这些虚拟的表对用户来说是透明的，但是只有最后一个虚拟的表才会被作为结果返回。如果没有在语句中指定某一个子句，那么将会跳过相应的步骤。    
![mysql执行顺序](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/mysql%E6%89%A7%E8%A1%8C%E9%A1%BA%E5%BA%8F.png)

下面我们来具体分析一下查询处理的每一个阶段:
1. FORM: 对FROM的左边的表和右边的表计算笛卡尔积。产生虚表VT1
2. ON: 对虚表VT1进行ON筛选，只有那些符合<join-condition>的行才会被记录在虚表VT2中。
3. JOIN： 如果指定了OUTER JOIN（比如left join、 right join），那么保留表中未匹配的行就会作为外部行添加到虚拟表VT2中，产生虚拟表VT3, rug from子句中包含两个以上的表的话，那么就会对上一个join连接产生的结果VT3和下一个表重复执行步骤1~3这三个步骤，一直到处理完所有的表为止。
4. WHERE： 对虚拟表VT3进行WHERE条件过滤。只有符合<where-condition>的记录才会被插入到虚拟表VT4中。
5. GROUP BY: 根据group by子句中的列，对VT4中的记录进行分组操作，产生VT5.
6. CUBE | ROLLUP: 对表VT5进行cube或者rollup操作，产生表VT6.
7. HAVING： 对虚拟表VT6应用having过滤，只有符合<having-condition>的记录才会被 插入到虚拟表VT7中。
8. SELECT： 执行select操作，选择指定的列，插入到虚拟表VT8中。
9. DISTINCT： 对VT8中的记录进行去重。产生虚拟表VT9.
10. ORDER BY: 将虚拟表VT9中的记录按照<order_by_list>进行排序操作，产生虚拟表VT10.
11. LIMIT：取出指定行的记录，产生虚拟表VT11, 并将结果返回。

# 总结
## OLTP(联机事务处理)系统和OLAP(联机分析处理)系统的区别
- OLTP(联机事务处理)系统：强调数据库内存效率，强调内存各种指标的命令率，强调绑定变量，强调并发操作，主要**应用在传统的关系型数据库**；
- OLAP(联机分析处理)系统：则强调数据分析，强调SQL执行时长，强调磁盘I/O，强调分区等，主要**应用在数据仓库系统**。



# 参考文献
[数据库存储引擎](https://github.com/jaywcjlove/mysql-tutorial/blob/master/chapter3/3.5.md)   
[MYSQL-索引](https://segmentfault.com/a/1190000003072424)   
[面试总结之数据库](http://hcxblog.com/2016/11/07/Interview-summary/)   
[MySQL的语句执行顺序](http://www.cnblogs.com/rollenholt/p/3776923.html)    
[B+树比B树更适合做文件索引的原因](https://blog.csdn.net/mine_song/article/details/63251546)   
[浅谈MySQL的B树索引与索引优化](https://juejin.im/post/5ab857675188255570060069)     
[Mysql索引优化](https://segmentfault.com/a/1190000009717352)
