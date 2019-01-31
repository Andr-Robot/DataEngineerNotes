* [Linux 统计某个字符串出现的次数](#linux-统计某个字符串出现的次数)
* [输出某文件夹下最大的文件](#输出某文件夹下最大的文件)
* [怎么修改主机名](#怎么修改主机名)
    * [查看主机名](#查看主机名)
    * [临时修改主机名](#临时修改主机名)
    * [永久修改主机名](#永久修改主机名)
    * [/etc/hostname 与 /etc/hosts 的区别](#etchostname-与-etchosts-的区别)
* [Linux五个查找命令的区别](#linux五个查找命令的区别)

# Linux 统计某个字符串出现的次数

```shell
grep -o objStr  filename|wc -l
```

**wc命令**：用来计算数字。利用wc指令我们可以计算文件的Byte数、字数或是列数，若不指定文件名称，或是所给予的文件名为“-”，则wc指令会从标准输入设备读取数据。

```
语法：
wc(选项)(参数)
选项：
-c或--bytes或——chars：只显示Bytes数；
-l或——lines：只显示列数；
-w或——words：只显示字数。
```

**grep**（global search regular expression(RE) and print out the line，全面搜索正则表达式并把行打印出来）是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。

```
-o 只输出文件中匹配到的部分。
```

# 输出某文件夹下最大的文件
首先查看文件夹下所有文件和文件夹的大小，只读取一层。

```
whicoder@debian:~/java/jdk$ du -a --max-depth=1
4	./README.html
4	./LICENSE
5740	./db
2020	./man
144	./THIRDPARTYLICENSEREADME.txt
4	./COPYRIGHT
108	./THIRDPARTYLICENSEREADME-JAVAFX.txt
4	./release
212248	./jre
792	./bin
208	./include
20608	./src.zip
135832	./lib
5084	./javafx-src.zip
382804	.
```
然后对读出的数据排序

```
whicoder@debian:~/java/jdk$ du -am --max-depth=1 | sort -nr
374	.
208	./jre
133	./lib
21	./src.zip
6	./db
5	./javafx-src.zip
2	./man
1	./THIRDPARTYLICENSEREADME.txt
1	./THIRDPARTYLICENSEREADME-JAVAFX.txt
1	./release
1	./README.html
1	./LICENSE
1	./include
1	./COPYRIGHT
1	./bin
```

最后结果：
```
whicoder@debian:~/java/jdk$ du -am --max-depth=1 | sort -n -r | head -n 2 | sort -n | head -1
208	./jre
```

# 怎么修改主机名
## 查看主机名
hostname 或 uname –n

## 临时修改主机名
使用`hostname`命令。

```
hostname 新主机名
```

```
mm@ubuntu:/etc$ sudo hostname vv
[sudo] password for mm: 
mm@ubuntu:/etc$ hostname
vv
mm@ubuntu:/etc$ cat hostname 
ubuntu
```

这样主机名字就临时被修改为`vv`，但是终端下不会立即显示生效后的主机名，重开一个终端窗口(通过ssh连接的终端需要重新连接才可以);

## 永久修改主机名
在 Ubuntu 系统中永久修改主机名也比较简单。主机名存放在 `/etc/hostname` 文件中，修改主机名时，编辑 `hostname` 文件，在文件中输入新的主机名并保存该文件即可。重启系统后，参照上面介绍的快速查看主机名的办法来确认主机名有没有修改成功。   
**注意：** 在其它Linux发行版中，并非都存在 `/etc/hostname` 文件。

## /etc/hostname 与 /etc/hosts 的区别
`/etc/hostname` 中存放的是主机名，`hostname` 文件的一个例子：

```
v-jiwan-ubuntu-temp
```

`/etc/hosts` 存放的是域名与 ip 的对应关系，是Linux系统中一个负责IP地址与域名快速解析的文件，域名与主机名没有任何关系，你可以为任何一个IP指定任意一个名字。

hosts文件是文本文件,每个地址映射占一行.每行的格式如下:

```
IP地址   主机或者域名   [主机的别名] [主机的别名]....
```

其中IP地址和主机是必需的，后面可以跟一个或多个别名，不同字段之间用一个或者多个空格(或TAB)分隔开。

`hosts` 文件的一个例子：

```
127.0.0.1       localhost
127.0.1.1       v-jiwan-ubuntu
```

# Linux五个查找命令的区别
参见[Linux的五个查找命令：find,locate,whereis,which,type 及其区别](https://www.cnblogs.com/kex1n/p/5233821.html)
