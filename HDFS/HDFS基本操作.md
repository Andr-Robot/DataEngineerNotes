<!-- GFM-TOC -->
* [常用命令](#常用命令)
* [Java API](#Java-API)
    * [使用FileSystem访问HDFS文件系统和本地文件系统(get/getLocal)](#使用filesystem访问hdfs文件系统和本地文件系统getgetlocal)
    * [创建文件(`create`)](#创建文件create)
    * [创建目录(`mkdirs`)](#创建目录mkdirs)
    * [检查目录或文件是否存在(`exists`)](#检查目录或文件是否存在exists)
    * [查看文件系统中文件元数据(`getFileStatus`)](#查看文件系统中文件元数据getFileStatus)
    * [查看目录下文件和目录列表(`listStatus`)](#查看目录下文件和目录列表listStatus)
    * [将本地文件写入到HDFS中(`copyBytes`)](#将本地文件写入到HDFS中copyBytes)
* [参考文献](#参考文献)
<!-- GFM-TOC -->

# 常用命令
**命令基本格式:** `hadoop fs -cmd < args >`
- 列出根目录下的文件和文件夹：`hadoop fs -ls /`
- 列出hdfs文件系统所有的目录和文件：`hadoop fs -ls -R /
`
- 创建文件夹：
    - `hadoop fs -mkdir /user/wcdoc`   
    只能一级一级的建目录，父目录不存在的话使用这个命令会报错；
    - `hadoop fs -mkdir -p < hdfs path> `   
    所创建的目录如果父目录不存在就创建该父目录
- 上传文件:
    - `hadoop fs -put < local file > < hdfs file >`    
    hdfs file的父目录一定要存在，否则命令不会执行
    - `hadoop fs -put  < local file or dir >...< hdfs dir >`    
    hdfs dir 一定要存在，否则命令不会执行，如：`hadoop fs -put /tmp/LICENSE.txt /user/wcdoc`
    - `hadoop fs -put - < hdsf  file>`   
    从键盘读取输入到hdfs file中，按Ctrl+D结束输入，hdfs file不能存在，否则命令不会执行
- 下载文件: 
    - `hadoop fs -get < hdfs file > < local file or dir>`   
    local file不能和 hdfs file名字不能相同，否则会提示文件已存在，没有重名的文件会复制到本地，如：`hadoop fs -get /user/wcdoc/LICENSE.txt /tmp`
    - `hadoop fs -get < hdfs file or dir > ... < local  dir >`   
    拷贝多个文件或目录到本地时，本地要为文件夹路径
- 查看文件：`hadoop fs -cat /user/wcdoc/LICENSE.txt`
- 删除文件(夹)：`hadoop fs -rm(r) /usr/wcdoc/LICENSE.txt`
- 列出指定目录下面每个文件大小：`hadoop fs -du /input`
- 列出指定目录或文件的占用的总大小：`hadoop fs -dus /input`
- 查询文件夹的磁盘空间限额和文件数目限额：`hadoop fs -count -q /tmp`


# Java API
## 使用FileSystem访问HDFS文件系统和本地文件系统(`get/getLocal`)
我们可以在Hadoop中使用FileSystem API来打开一个文件的输入流，然后我们可以对文件进行各种的操作实现。Configuration 对象封装了客户端或服务器的配置，通过设置配置文件读取类路径来实现。
```java
// 返回的是默认文件系统（在core-site.xml中指定的，如果没有指定，则使用默认的本地文件系统）。 
public statis FileSystem get(Configuration conf) throws IOException
// 通过给定的URI方法和权限来确定要使用的文件系统，如果给定的URI中没有指定方案，则返回默认的文件系统。
public statis FileSystem get(URI uri , Configuration conf) throws IOException
// 添加了user参数，顾名思义就是让给定用户来访问文件系统
public statis FileSystem get(URI uri , Configuration conf ,String user) throws IOException
// 获取本地文件系统
public static LocalFileSystem getLocal(Configuration conf) throws IOException
```

## 创建文件(`create`)

```java
Configuration conf = new Configuration();
FileSystem fs = FileSystem.get(conf);
fs.create(new Path(hdfsPath));
```

## 创建目录(`mkdirs`)

```java
Configuration conf = new Configuration();
FileSystem fs = FileSystem.get(conf);
fs.mkdirs(new Path(hdfsPath));
```

和上边的create方法一样，都会根据path建立相应的文件或目录，如果父级目录不存在，则自动创建。   

## 检查目录或文件是否存在(`exists`)

```java
Configuration conf = new Configuration();
FileSystem fs = FileSystem.get(conf);
fs.exists(new Path(hdfsPath));
```

## 查看文件系统中文件元数据(`getFileStatus`)
包含文件长度、块大小、备份、修改时间、所有者以及权限信息。

```java
public class getStatus {
 
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		FileSystem fs = FileSystem.get(conf);
		FileStatus file = fs.getFileStatus(new Path(filepath));
		long len = file.getLen();                   //文件长度
		String pathSource = file.getPath().toString();//文件路径
		String fileName = file.getPath().getName();   // 文件名称
		String parentPath = file.getPath().getParent().toString();//文件父路径
		Timestamp timestamp = new Timestamp(file.getModificationTime());//文件最后修改时间
		long blockSize = file.getBlockSize();   //文件块大小
		String group = file.getGroup();         //文件所属组
		String owner = file.getOwner();          // 文件拥有者
		long accessTime = file.getAccessTime();  //该文件上次访问时间
		short replication = file.getReplication(); //文件副本数
		System.out.println("文件长度: "+len+"\n"+ 
		    "文件路径: "+pathSource+"\n"+
		    "文件名称: "+fileName+"\n"+
		    "文件父路径: "+parentPath+"\n"+
		    "文件最后修改时间: "+timestamp+"\n"+
		    "文件块大小: "+blockSize+"\n"+
		    "文件所属组: "+group+"\n"+
		    "文件拥有者: "+owner+"\n"+
		    "该文件上次访问时间: "+accessTime+"\n"+
		    "文件副本数: "+replication+"\n");
	}
}
```

FileStatus有一个isFile()方法，能够判断是否为文件或是否存在；file.isDirectory()用来判断是否为文件夹或是否存在。如果判断是否存在使用exists方法比较方便。

## 查看目录下文件和目录列表(`listStatus`)

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileStatus;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
 
public class getPaths {
 
	public static void main(String[] args) throws Exception {
 
		Configuration conf = new Configuration();
		FileSystem fs = FileSystem.get(conf);
		FileStatus[] status = fs.listStatus(new Path(hdfspath));
		for (int i = 0; i < status.length; i++) {
		    System.out.println(status[i].getPath().toString());
		}
	}
}
```

## 将本地文件写入到HDFS中(`copyBytes`)

```java
import java.io.BufferedInputStream;
import java.io.FileInputStream;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.URI;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;

/**
 *将本地文件系统的文件通过java-API写入到HDFS文件
 */
public class FileCopyFromLocal {
    public static void main(String[] args) throws Exception {
        String source="/usr/local/filecontent/demo";//linux中的文件路徑,demo存在一定数据
        String destination="hdfs://......";//HDFS的路徑
        InputStream in = new BufferedInputStream(new FileInputStream(source));
        //HDFS读写的配置文件
        Configuration conf = new Configuration();
        //调用Filesystem的create方法返回的是FSDataOutputStream对象
        //该对象不允许在文件中定位，因为HDFS只允许一个已打开的文件顺序写入或追加
        FileSystem fs = FileSystem.get(URI.create(destination),conf);
        OutputStream out = fs.create(new Path(destination));
        IOUtils.copyBytes(in, out, 4096, true);
    }
}
```


# 参考文献
[hadoop HDFS常用文件操作命令](https://segmentfault.com/a/1190000002672666#articleHeader6)       
[Hadoop Shell命令——官方文档](https://hadoop.apache.org/docs/r1.0.4/cn/hdfs_shell.html)    
[利用HDFS的JavaAPI编程[三]](http://blog.pureisle.net/archives/1806.html)   
[HDFS文件系统的JAVA-API操作(一)](http://www.cnblogs.com/jackchen-Net/p/6588662.html)    
[调用JAVA API 对 HDFS 进行文件的读取、写入、上传、下载、删除等操作](https://blog.csdn.net/DF_XIAO/article/details/50601727)   
[利用HDFS的JavaAPI编程[一]](http://blog.pureisle.net/archives/1701.html)    
[利用HDFS的JavaAPI编程[二]](http://blog.pureisle.net/archives/1794.html)   
[Hadoop FIleSystem API JAVA操作](https://blog.csdn.net/Leafage_M/article/details/75041335)    
[HDFS 常用命令](http://blog.sanyuehua.net/2017/11/01/Hadoop-HDFS/)