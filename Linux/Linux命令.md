* [grep](#grep)
* [wc](#wc)
* [df](#df)
* [du](#du)
* [sort](#sort)
* [head](#head)
* [创建文件的命令](#创建文件的命令)
    * [vi/vim](#vi/vim)
    * [gedit](#gedit)
    * [touch](#touch)
    * [cat](#cat)
    * [echo](#echo)
* [ps](#ps)
* [kill](#kill)
* [free](#free)
* [top](#top)
* [netstat](#netstat)
* [find](#find)
* [crontab](#crontab)
* [压缩](#压缩)
    * [gzip](#gzip)
    * [tar](#tar)
    * [lzop](#lzop)
* [chmod](#chmod)
* [chown](#chown)
* [参考文献](#参考文献)

# grep
它能使用正则表达式搜索文本，并把匹配的行打印出来。
1. -c 或 --count : 计算符合样式的列数。
2. -v 或 --revert-match : 显示不包含匹配文本的所有行。
3. -n 或 --line-number : 在显示符合样式的那一行之前，标示出该行的列数编号。
4. -o 只输出文件中匹配到的部分。


```
mm@ubuntu:~$ cat file1 
toutiao
sdfasdf
toutiao
fdasfafdsf
fgfggfg
mm@ubuntu:~$ grep toutiao file1 
toutiao
toutiao
mm@ubuntu:~$ grep -o toutiao file1 
toutiao
toutiao
mm@ubuntu:~$ grep -c toutiao file1 
2
mm@ubuntu:~$ grep -n toutiao file1 
1:toutiao
3:toutiao
mm@ubuntu:~$ grep -o toutiao file1 | wc -l
2
mm@ubuntu:~$ grep -v toutiao file1 
sdfasdf
fdasfafdsf
fgfggfg
```

# wc
wc命令用来计算数字。利用wc指令我们可以计算文件的Byte数、字数或是列数，若不指定文件名称，或是所给予的文件名为“-”，则wc指令会从标准输入设备读取数据。

- **语法**：wc(选项)(参数)
1. -c或--bytes或——chars：只显示Bytes数；
2. -l或——lines：只显示列数；
3. -w或——words：只显示字数。

```
mm@ubuntu:~$ cat file1 
toutiao dsfds
sdfasdf
toutiao
fdasfafdsf
fgfggfg
mm@ubuntu:~$ wc -l file1 
5 file1
mm@ubuntu:~$ wc -c file1 
49 file1
mm@ubuntu:~$ wc -w file1 
6 file1
```

# df
df命令用于显示磁盘分区上的可使用的磁盘空间。默认显示单位为KB。可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。    

- **语法**：df(选项)(参数)
- **参数**：文件：指定文件系统上的文件。
1. -m或--megabytes：指定区块大小为1048576字节；
2. -h或--human-readable：以可读性较高的方式来显示信息；
3. -H或--si：与-h参数相同，但在计算时是以1000 Bytes为换算单位而非1024 Bytes；
4. -k或--kilobytes：指定区块大小为1024字节；
5. -a：包含文件的大小

```
whicoder@debian:~$ df
文件系统          1K-块    已用     可用 已用% 挂载点
udev            1005652       0  1005652    0% /dev
tmpfs            203380    9616   193764    5% /run
/dev/sda1      49279636 6879284 39867388   15% /
whicoder@debian:~$ df -H
文件系统        容量  已用  可用 已用% 挂载点
udev            1.1G     0  1.1G    0% /dev
tmpfs           209M  9.9M  199M    5% /run
/dev/sda1        51G  7.1G   41G   15% /
whicoder@debian:~$ df -h
文件系统        容量  已用  可用 已用% 挂载点
udev            983M     0  983M    0% /dev
tmpfs           199M  9.4M  190M    5% /run
/dev/sda1        47G  6.6G   39G   15% /
whicoder@debian:~$ df -m
文件系统       1M-块  已用  可用 已用% 挂载点
udev             983     0   983    0% /dev
tmpfs            199    10   190    5% /run
/dev/sda1      48125  6720 38932   15% /
whicoder@debian:~$ df -k
文件系统          1K-块    已用     可用 已用% 挂载点
udev            1005652       0  1005652    0% /dev
tmpfs            203380    9616   193764    5% /run
/dev/sda1      49279636 6883652 39863020   15% /
```

# du
用于显示目录或文件的大小。du会显示指定的目录或文件所占用的磁盘空间。
- **语法**：du [选项][文件]
1. -b或-bytes 显示目录或文件大小时，以byte为单位。
2. -c或--total 除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和。
3. -k或--kilobytes 以KB(1024bytes)为单位输出。
4. -m或--megabytes 以MB为单位输出。
5. -s或--summarize 仅显示总计，只列出最后加总的值。
6. -h或--human-readable 以K，M，G为单位，提高信息的可读性。
7. --max-depth=<目录层数> 超过指定层数的目录后，予以忽略。

# sort
它将文件进行排序，并将排序结果标准输出。sort命令既可以从特定的文件，也可以从stdin中获取输入。
- **语法**：sort(选项)(参数)
- **参数**：文件：指定待排序的文件列表。

1. -b：忽略每行前面开始处的空格字符；
2. -c：检查文件是否已经按照顺序排序；
3. -d：排序时，处理英文字母、数字及空格字符外，忽略其他的字符；
4. -f：排序时，将小写字母视为大写字母；
5. -n：依照数值的大小排序；
6. -r：以相反的顺序来排序；
7. -o<输出文件>：将排序后的结果存入制定的文件；

# head
用于显示文件的开头的内容。在默认情况下，head命令显示文件的头10行内容。
- **语法**：head(选项)(参数)
- **参数**：文件列表：指定显示头部内容的文件列表。 
1. -n<数字>：指定显示头部内容的行数；

# 创建文件的命令
## vi/vim
使用vi/vim编辑文本，然后保存就可以了。

## gedit
使用gedit编辑文本，然后保存就可以了。

## touch
用于修改文件或者目录的时间属性，包括存取时间和更改时间。**若文件不存在，系统会建立一个新的文件**。

使用指令"touch"修改文件"testfile"的时间属性为当前系统时间，输入如下命令：

```
$ touch testfile                #修改文件的时间属性 
```

示例：

```
$ ls -l testfile                #查看文件的时间属性  
#原来文件的修改时间为16:09  
-rw-r--r-- 1 hdd hdd 55 2011-08-22 16:09 testfile  
$ touch testfile                #修改文件时间属性为当前系统时间  
$ ls -l testfile                #查看文件的时间属性  
#修改后文件的时间属性为当前系统时间  
-rw-r--r-- 1 hdd hdd 55 2011-08-22 19:53 testfile  
```

使用指令"touch"时，如果指定的文件不存在，则将创建一个新的空白文件。例如，在当前目录下，使用该指令创建一个空白文件"file"，输入如下命令：

```
$ touch file            #创建一个名为“file”的新的空白文件 
```

## cat
cat 命令用于连接文件并打印到标准输出设备上。    
可以使用`>`或`>>`将标准输入的内容重定向到文件中。   
**注意：`>`这个是覆盖，`>>`这个是追加。**

参数：
1. -n 或 --number：由 1 开始对所有输出的行数编号。
2. -b 或 --number-nonblank：和 -n 相似，只不过对于空白行不编号。

## echo
用于字符串的输出。

也可以和cat命令一样用于将字符串重定向到文件中。

# ps
用于显示当前进程 (process) 的状态。   

1. ps a 显示现行终端机下的所有程序，包括其他用户的程序。
2. ps -A 显示所有程序。
3. ps -e 此参数的效果和指定"A"参数相同。
4. ps -au 显示较详细的资讯
5. ps -aux 显示所有包含其他使用者的行程

用来查看某个服务的进程：

```
// 显示出所有的bash进程
whicoder@debian:~/java/jdk$ ps -aux | grep bash
whicoder   2057  0.0  0.2  21408  5324 pts/0    Ss   18:48   0:00 bash
whicoder  11863  0.0  0.0  12988   984 pts/0    S+   20:28   0:00 grep bash

// 显示出所有的bash进程，去处掉当前的grep进程。
whicoder@debian:~/java/jdk$ ps -ef | grep bash | grep -v grep
whicoder   2057   2052  0 18:48 pts/0    00:00:00 bash
```
# kill
用来删除执行中的程序或工作。kill可将指定的信息送至程序。预设的信息为SIGTERM(15),可将指定程序终止。若仍无法终止该程序，可使用SIGKILL(9)信息尝试强制删除程序。程序或工作的编号可利用ps指令或job指令查看。

**语法**：kill(选项)(参数)

**选项**：
1. -a：当处理当前进程时，不限制命令名和进程号的对应关系；
2. -l <信息编号>：若不加<信息编号>选项，则-l参数会列出全部的信息名称；
3. -p：指定kill 命令只打印相关进程的进程号，而不发送任何信号；
4. -s <信息名称或编号>：指定要送出的信息；
5. -u：指定用户。

**参数**：进程或作业识别号：指定要删除的进程或作业。

下面是常用的信号：

```
HUP     1    终端断线
INT     2    中断（同 Ctrl + C）
QUIT    3    退出（同 Ctrl + \）
TERM   15    终止
KILL    9    强制终止
CONT   18    继续（与STOP相反， fg/bg命令）
STOP   19    暂停（同 Ctrl + Z）
```

```
whicoder@debian:~/java/jdk$ ps -ef | grep gedit | grep -v grep
whicoder  12121   2057  0 20:39 pts/0    00:00:00 gedit 1
whicoder@debian:~/java/jdk$ kill 12121
whicoder@debian:~/java/jdk$ ps -ef | grep gedit | grep -v grep
whicoder@debian:~/java/jdk$ 
```

# free
可以显示当前系统未使用的和已使用的内存数目，还可以显示被内核使用的内存缓冲区。

**语法**：free(选项)

**选项**
1. -b：以Byte为单位显示内存使用情况；
2. -k：以KB为单位显示内存使用情况；
3. -m：以MB为单位显示内存使用情况；
4. -o：不显示缓冲区调节列；
5. -s<间隔秒数>：持续观察内存使用状况；
6. -t：显示内存总和列；
7. -V：显示版本信息。


```
whicoder@debian:~/java/jdk$ free -m
              total        used        free      shared  buff/cache   available
Mem:           1986         787          90          16        1108        1012
Swap:          2044           0        2044
```

**第一部分Mem行解释**：
- total：内存总数；
- used：已经使用的内存数；
- free：空闲的内存数；
- shared：当前已经废弃不用；


关系：total = used + free

**第二部分是指交换分区**

# top
可以实时动态地查看系统的整体运行情况。     
**可以用这个命令查看CPU的使用情况。**

# netstat
用来打印Linux中网络系统的状态信息，可让你得知整个Linux系统的网络情况。    

**语法**：netstat(选项)   

**选项**：
1. -a或--all：显示所有连线中的Socket；
2. -l或--listening：显示监控中的服务器的Socket；
3. -c或--continuous：持续列出网络状态；
4. -t或--tcp：显示TCP传输协议的连线状况；
5. -u或--udp：显示UDP传输协议的连线状况；
6. -r或--route：显示Routing Table；

往往某些服务启动不了，是所需端口被占用了，如果我们需要知道2809号端口的情况，我们可以这样，如下命令：   

```
netstat -pan|grep 2809
```

查看所有已经建立的连接

```
netstat -antp
```

查看网络统计信息

```
netstat -s
```

# find
用来在指定目录下查找文件。任何位于参数之前的字符串都将被视为欲查找的目录名。如果使用该命令时，不设置任何参数，则find命令将在当前目录下查找子目录与文件。并且将查找到的子目录和文件全部进行显示。

**语法**：find(选项)(参数)

**选项**：
- -maxdepth<目录层级>：设置最大目录层级；
- -mindepth<目录层级>：设置最小目录层级；
- -name<范本样式>：指定字符串作为寻找文件或目录的范本样式；

**参数**：    
起始目录：查找文件的起始目录。

列出当前目录及子目录下所有文件和文件夹：

```
find
```

在/home目录下查找以.gz结尾的文件名：

```
whicoder@debian:~$ find /home -name "*.gz"
/home/whicoder/elasticsearch-6.2.4.tar.gz
/home/whicoder/logstash-6.2.4.tar.gz
/home/whicoder/java/jdk/lib/missioncontrol/p2/org.eclipse.equinox.p2.engine/profileRegistry/JMC.profile/1513160564559.profile.gz
/home/whicoder/java/jdk/lib/missioncontrol/p2/org.eclipse.equinox.p2.engine/profileRegistry/JMC.profile/1513160551742.profile.gz
/home/whicoder/java/jdk/lib/missioncontrol/p2/org.eclipse.equinox.p2.engine/profileRegistry/JMC.profile/1513160551369.profile.gz
/home/whicoder/java/jdk/lib/missioncontrol/p2/org.eclipse.equinox.p2.engine/profileRegistry/JMC.profile/1513160561556.profile.gz
/home/whicoder/kibana-6.2.4-linux-x86_64.tar.gz
/home/whicoder/.cache/vmware/drag_and_drop/R5YZdF/jdk-8u171-linux-x64.tar.gz
/home/whicoder/.cache/vmware/drag_and_drop/9e9M07/elasticsearch-6.2.4.tar.gz
/home/whicoder/.cache/vmware/drag_and_drop/9e9M07/logstash-6.2.4.tar.gz
/home/whicoder/.cache/vmware/drag_and_drop/9e9M07/kibana-6.2.4-linux-x86_64.tar.gz
/home/whicoder/.cache/vmware/drag_and_drop/sTLvNx/jdk-8u171-linux-x64.tar.gz
```

寻找当前目录向下最大深度限制为1搜索：
```
whicoder@debian:~$ find /home/whicoder/ -maxdepth 1 -name "*.gz"
/home/whicoder/elasticsearch-6.2.4.tar.gz
/home/whicoder/logstash-6.2.4.tar.gz
/home/whicoder/kibana-6.2.4-linux-x86_64.tar.gz
```

# crontab
用来提交和管理用户的需要周期性执行的任务。
**语法**：crontab(选项)(参数)    
**选项**：
- -e：编辑该用户的计时器设置；
- -l：列出该用户的计时器设置；
- -r：删除该用户的计时器设置；
- -u<用户名称>：指定要设定计时器的用户名称。

**参数**：   
crontab文件：指定包含待执行任务的crontab文件。

crontab文件的含义：用户所建立的crontab文件中，每一行都代表一项任务，每行的每个字段代表一项设置，它的格式共分为六个字段，前五段是时间设定段，第六段是要执行的命令段，格式如下：

```
minute   hour   day   month   week   command     
顺序：分 时 日 月 周
```

- minute： 表示分钟，可以是从0到59之间的任何整数。
- hour：表示小时，可以是从0到23之间的任何整数。
- day：表示日期，可以是从1到31之间的任何整数。
- month：表示月份，可以是从1到12之间的任何整数。
- week：表示星期几，可以是从0到7之间的任何整数，这里的0或7代表星期日。
- command：要执行的命令，可以是**系统命令**，也可以是自己编写的**脚本文件**。

在以上各个字段中，还可以使用以下特殊字符：
- 星号（*）：代表所有可能的值，例如month字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。
- 逗号（,）：可以用逗号隔开的值指定一个列表范围，例如，“1,2,5,7,8,9”
- 中杠（-）：可以用整数之间的中杠表示一个整数范围，例如“2-6”表示“2,3,4,5,6”
- 正斜线（/）：可以用正斜线指定时间的间隔频率，例如“0-23/2”表示每两小时执行一次。同时正斜线可以和星号一起使用，例如*/10，如果用在minute字段，表示每十分钟执行一次。

使用案例：   
1. 键入`crontab  -e`编辑crontab服务文件
2. 查看该用户下的crontab服务是否创建成功，用`crontab  -l`命令  

# 压缩
## gzip
gzip命令用来压缩文件。gzip是个使用广泛的压缩程序，文件经它压缩过后，其名称后面会多处“.gz”扩展名。   
**语法**：gzip(选项)(参数)   
**选项**：
- -d或--decompress或----uncompress：解开压缩文件；
- -f或——force：强行压缩文件。不理会文件名称或硬连接是否存在以及该文件是否为符号连接；
- -l或——list：列出压缩文件的相关信息；
- -r或——recursive：递归处理，将指定目录下的所有文件及子目录一并处理；

**参数**：   
文件列表：指定要压缩的文件列表。

## tar
tar命令可以为linux的文件和目录创建。利用tar命令，可以把一大堆的文件和目录全部打包成一个文件，这对于备份文件或将几个文件组合成为一个文件以便于网络传输是非常有用的。

首先要弄清两个概念：**打包和压缩**。打包是指将一大堆文件或目录变成一个总的文件；压缩则是将一个大的文件通过一些压缩算法变成一个小文件。

**Linux中很多压缩程序只能针对一个文件进行压缩**，这样当你想要压缩一大堆文件时，你得先将这一大堆文件先打成一个包（tar命令），然后再用压缩程序进行压缩（gzip bzip2命令）。    

**语法**：tar(选项)(参数)
**选项**：
- -c或--create：建立新的备份文件；
- -C <目录>：这个选项用在解压缩，若要在特定目录解压缩，可以使用这个选项。
- -x或--extract或--get：从备份文件中还原文件；
- -t或--list：列出备份文件的内容；
- -z或--gzip或--ungzip：通过gzip指令处理备份文件；
- Z或--compress或--uncompress：通过compress指令处理备份文件；
- -f<备份文件>或--file=<备份文件>：指定备份文件；
- -r：添加文件到已经压缩的文件；
- -v：显示操作过程；
- -j：支持bzip2解压文件；

**参数**：   
文件或目录：指定要打包的文件或目录列表。


```
// 将文件全部打包成tar包
tar -cvf log.tar log2012.log    仅打包，不压缩！ 
tar -zcvf log.tar.gz log2012.log   打包后，以 gzip 压缩 
tar -jcvf log.tar.bz2 log2012.log  打包后，以 bzip2 压缩 

// 将tar包解压缩
tar -zxvf /opt/soft/test/log.tar.gz
```

## lzop
**无损压缩**    
lzop工具最适合在注重压缩速度的场合，压缩文件时会新建.lzo文件，而原文件保持不变(使用-U选项除外)    

```
# lzop -v test # 创建test.lzo压缩文件，输出详细信息，保留test文件不变

# lzop -Uv test # 创建test.lzo压缩文件，输出详细信息，删除test文件

# lzop -t test.lzo # 测试test.lzo压缩文件的完整性

# lzop –info test.lzo # 列出test.lzo中各个文件的文件头

# lzop -l test.lzo # 列出test.lzo中各个文件的压缩信息

# lzop –ls test.lzo # 列出test.lzo文件的内容，同ls -l功能

# cat test | lzop > t.lzo # 压缩标准输入并定向到标准输出

# lzop -dv test.lzo # 解压test.lzo得到test文件，输出详细信息，保留test.lzo不变
```


# chmod
chmod命令用来变更**文件或目录的权限**。

![chmod](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/chmod.gif)     
- r=读取属性　　//值＝4
- w=写入属性　  //值＝2
- x=执行属性　  //值＝1


```
chmod u+x,g+w f01　　//为文件f01设置自己可以执行，组员可以写入的权限
chmod u=rwx,g=rw,o=r f01
chmod 764 f01
chmod a+x f01　　//对文件f01的u,g,o都设置可执行属性
```

# chown
chown命令**改变某个文件或目录的所有者和所属的组**，该命令可以向某个用户授权，使该用户变成指定文件的所有者或者改变文件所属的组。

```
# 将目录/usr/meng及其下面的所有文件、子目录的文件主改成 liu：
chown -R liu /usr/meng
```


# 参考文献
[Linux命令大全](http://man.linuxde.net/)