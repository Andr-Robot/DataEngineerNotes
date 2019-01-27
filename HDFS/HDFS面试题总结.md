<!-- GFM-TOC -->
* [HDFS读写流程](#HDFS读写流程)
* [namenode宕机怎么处理](#namenode宕机怎么处理)
* [HDFS查看压缩文件中的内容](#HDFS查看压缩文件中的内容)
    * [lzo文件(先按照lzop命令)](#lzo文件(先按照lzop命令))
    * [gz压缩](#gz压缩)
    * [-cat在终端显示文件内容](#-cat在终端显示文件内容)
    * [-text](#-text)
* [查看HDFS文件夹下文件或文件夹的大小](#查看HDFS文件夹下文件或文件夹的大小)
    * [-count列出文件夹数量、文件数量、内容大小](#-count列出文件夹数量文件数量内容大小)
    * [-du显示文件大小](#-du显示文件大小)
    * [理解hadoop fsck、fs -dus、-count -q的大小输出](#理解hadoop fsck、fs -dus、-count -q的大小输出)
        * [为什么逻辑空间一般不等于物理空间？](#为什么逻辑空间一般不等于物理空间？)
        * [hadoop fsck和hadoop fs -dus](#hadoop fsck和hadoop fs -dus)
        * [hadoop fs -count -q](#hadoop fs -count -q)
* [HDFS查看文件的前几行、后几行和行数](#HDFS查看文件的前几行、后几行和行数)
* [hdfs create一个文件的流程](#hdfs create一个文件的流程)
<!-- GFM-TOC -->


# HDFS读写流程

# namenode宕机怎么处理
1. Secondary NameNode：恢复时间慢，会有部分数据丢失。（edit\_logs和fs\_Image）
2. StandBy NameNode：信息不会丢失，恢复快（秒级），部署简单

# HDFS查看压缩文件中的内容
## lzo文件(先按照lzop命令) 

```shell
hadoop fs -cat /user/2017-03-06/part-r-00255.lzo | lzop -d | head -1
```

`lzop -d`：解压

## gz压缩

```shell
hadoop fs -cat /tmp/temp.txt.gz | gzip -d

hadoop fs -cat /tmp/temp.txt.gz | zcat
```

- `gzip -d`：解开压缩文件
- `zcat`：用于不真正解压缩文件，就能显示压缩包中文件的内容的场合。

```
-cat	-cat	查看文件内容	hadoop fs -cat /input/abc.txt
-text	-text	查看文件或者 zip 的内容	hadoop fs -text /input/abc.txt
```

## -cat在终端显示文件内容

```
hdfs dfs -cat /two.txt
hdfs dfs -cat /input/*  # 查看指定目录下面所有的文件的内容
```

## -text
在终端显示文件内容，将源文件输出为文本格式。允许的格式是zip和TextRecordInputStream。

```
hdfs dfs -text /input/yarn-site.xml
 ./hdfs dfs -text /demodemo.zip  (有可能会输入一堆乱码)
```

# 查看HDFS文件夹下文件或文件夹的大小
这里应该使用：

```
hadoop fs -du hdfs://172.16.0.226:8020/test/sys_dict/ | sort -nr | head -n 1
```

- `sort -nr`表示按数字大小倒序排列，其中`-n`表示按数字大小排序，`-r`表示倒序排序
- `head -n 1`表示显示前几行


```
-count	-count [-q] <路径>	查询文件夹的磁盘空间限额和文件数目限额	hadoop fs -count -p /tmp
-du	-du <路径>	统计目录下文件的大小	hadoop fs -du /input
-dus	-dus <路径>	汇总统计目录下文件和文件夹的大小	hadoop fs -du /
```

## -count列出文件夹数量、文件数量、内容大小

```
hdfs dfs -count -q /
```

## -du显示文件大小

```
hdfs dfs -du /input #列出指定目录下面每个文件大小
hdfs dfs -du -s /input #列出指定目录或文件的占用的总大小
```

## 理解hadoop fsck、fs -dus、-count -q的大小输出
参见[理解hadoop fsck、fs -dus、-count -q的大小输出](http://www.opstool.com/article/255)
- **逻辑空间**：即分布式文件系统上真正的文件大小
- **物理空间**：即存在分布式文件系统上该文件实际占用的空间

### 为什么逻辑空间一般不等于物理空间？
分布式文件系统为了保证文件的可靠性，往往会保存多个备份（一般是3份)，只要备份数不为1的情况下，一般物理空间会是逻辑空间的几倍。关系如下：
> HDFS物理空间 = 逻辑空间 * block备份数

### hadoop fsck和hadoop fs -dus 
执行`hadoop fsck`和`hadoop fs -dus`显示的文件大小表示的是文件占用的逻辑空间。

```shell
$ hadoop fsck /path/to/directory
 Total size:    16565944775310 B    <=== 看这里
 Total dirs:    3922
 Total files:   418464
 Total blocks (validated):      502705 (avg. block size 32953610 B)
 Minimally replicated blocks:   502705 (100.0 %)
 Over-replicated blocks:        0 (0.0 %)
 Under-replicated blocks:       0 (0.0 %)
 Mis-replicated blocks:         0 (0.0 %)
 Default replication factor:    3
 Average block replication:     3.0
 Corrupt blocks:                0
 Missing replicas:              0 (0.0 %)
 Number of data-nodes:          18
 Number of racks:               1
FSCK ended at Thu Oct 20 20:49:59 CET 2011 in 7516 milliseconds
 
The filesystem under path '/path/to/directory' is HEALTHY

$ hadoop fs -dus /path/to/directory
hdfs://master:54310/path/to/directory        16565944775310    <=== 看这里
```

### hadoop fs -count -q 
通过执行`hadoop fs -count -q /path/to/directory` 可以看到这个目录真正的空间使用情况。执行结果如下:

```
$ hadoop fs -count -q /path/to/directory
  QUOTA  REMAINING_QUOTA     SPACE_QUOTA  REMAINING_SPACE_QUOTA    DIR_COUNT  FILE_COUNT      CONTENT_SIZE FILE_NAME
   none              inf  54975581388800          5277747062870        3922       418464    16565944775310 hdfs://master:54310/path/to/directory
```
fs -count -q会输出8列，分别表示如下:
1. 命名空间的quota（限制文件数）	
2. 剩余的命名空间quota	
3. 物理空间的quota （限制空间占用大小）	
4. 剩余的物理空间	
5. 目录数统计	
6. 文件数统计	
7. 目录逻辑空间总大小	
8. 路径

可以看出通过`hadoop fs -count -q` 可以看到一个目录比较详细的空间和qutoa占用情况，包含了物理空间、逻辑空间、文件数、目录数、qutoa剩余量等。


# HDFS查看文件的前几行、后几行和行数
- 随机返回指定行数的样本数据 

```shell
hadoop fs -cat /test/gonganbu/scene_analysis_suggestion/* | shuf -n 5
```

- 返回前几行的样本数据 

```shell
hadoop fs -cat /test/gonganbu/scene_analysis_suggestion/* | head -100
```

- 返回最后几行的样本数据 

```shell
hadoop fs -cat /test/gonganbu/scene_analysis_suggestion/* | tail -5
```

- 查看文本行数 

```shell
hadoop fs -cat hdfs://172.16.0.226:8020/test/sys_dict/sysdict_case_type.csv |wc -l
```

- 查看文件大小(单位byte) 

```shell
hadoop fs -du hdfs://172.16.0.226:8020/test/sys_dict/*

hadoop fs -count hdfs://172.16.0.226:8020/test/sys_dict/*
```

# hdfs create一个文件的流程
参见[HDFS写文件过程分析](http://shiyanjun.cn/archives/942.html)
1. Client调用DistributedFileSystem对象的create方法，创建一个文件输出流（FSDataOutputStream）对象
2. 通过DistributedFileSystem对象与Hadoop集群的NameNode进行一次RPC远程调用，在HDFS的Namespace中创建一个文件条目（Entry），该条目没有任何的Block
3. 通过FSDataOutputStream对象，向DataNode写入数据，数据首先被写入FSDataOutputStream对象内部的Buffer中，然后数据被分割成一个个Packet数据包
4. 以Packet最小单位，基于Socket连接发送到按特定算法选择的HDFS集群中一组DataNode（正常是3个，可能大于等于1）中的一个节点上，在这组DataNode组成的Pipeline上依次传输Packet
5. 这组DataNode组成的Pipeline反方向上，发送ack，最终由Pipeline中第一个DataNode节点将Pipeline ack发送给Client
6. 完成向文件写入数据，Client在文件输出流（FSDataOutputStream）对象上调用close方法，关闭流
7. 调用DistributedFileSystem对象的complete方法，通知NameNode文件写入成功


