* [主程序](#主程序)
* [map程序](#map程序)
* [sort程序](#sort程序)
* [reduce程序](#reduce程序)
* [测试文件](#测试文件)
* [执行结果](#执行结果)
* [参考文献](#参考文献)

# 主程序
这里的测试文件是放在项目的根目录下，也可以自己制定一个文件。

```python
# MapReduceJob.py
if __name__ == '__main__':
    filepath = "test"
    lines = open(filepath, 'r').readlines()
    for line in lines:
        print(line.strip())
```

# map程序

```python
# myMap.py
import sys

def mapword(w):
    return w, 1

if __name__ == '__main__':
    for line in sys.stdin:
        line = line.strip()
        words = line.split()
        m = map(mapword, words)
        for word, count in list(m):
            print('%s\t%d' % (word, count))
```

# sort程序

```python
# mySort.py
import sys

if __name__ == '__main__':
    m = []
    for line in sys.stdin:
        line = line.strip()
        word, count = line.split('\t')
        m.append((word, count))
    m = sorted(m)
    for i in m:
        print('%s\t%s' % i)
```

# reduce程序

```python
# myReduce.py
import sys

if __name__ == '__main__':
    current_word = None
    current_count = 0
    word = None

    for line in sys.stdin:
        line = line.strip()
        word, count = line.split('\t')
        count = int(count)
        if current_word == word:
            current_count += count
        else:
            if current_word:
                print('%s\t%s' % (current_word, current_count))
            current_word = word
            current_count = count
    if current_word == word:
        print('%s\t%s' % (current_word, current_count))
```

# 测试文件

```
a b c d
a b c d
aa bb cc dd
ee ff gg hh
```

# 执行结果
在Windows下，通过`cmd`运行上面的代码。

```
E:\Project\pycharm\MapReduceDemo>python MapReduceJob.py
a b c d
a b c d
aa bb cc dd
ee ff gg hh

E:\Project\pycharm\MapReduceDemo>python MapReduceJob.py | python myMap.py
a       1
b       1
c       1
d       1
a       1
b       1
c       1
d       1
aa      1
bb      1
cc      1
dd      1
ee      1
ff      1
gg      1
hh      1

E:\Project\pycharm\MapReduceDemo>python MapReduceJob.py | python myMap.py | python mySort.py
a       1
a       1
aa      1
b       1
b       1
bb      1
c       1
c       1
cc      1
d       1
d       1
dd      1
ee      1
ff      1
gg      1
hh      1

E:\Project\pycharm\MapReduceDemo>python MapReduceJob.py | python myMap.py | python mySort.py | python myRecuce.py
a       2
aa      1
b       2
bb      1
c       2
cc      1
d       2
dd      1
ee      1
ff      1
gg      1
hh      1
```


# 参考文献
[用python模拟hadoop的map reduce过程](http://abloz.com/%E6%8A%80%E6%9C%AF/2012/06/25/python-simulation-hadoop-map-the-reduce-process/)    
[用python模拟hadoop的map reduce过程](https://ablozhou.wordpress.com/2012/06/25/python-simulation-hadoop-map-the-reduce-process/)