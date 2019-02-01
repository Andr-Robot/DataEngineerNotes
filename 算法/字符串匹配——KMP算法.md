* [KMP（未优化）版本实现](#kmp未优化版本实现)
* [KMP找到所有匹配的子串的起始下标](#kmp找到所有匹配的子串的起始下标)
* [参考文献](#参考文献)

# KMP（未优化）版本实现
代码实现：      
这个版本是，返回第一个匹配的字符串的起始下标。

```python
# 获取 最长相同真前后缀长度数组
def getNextList(p):
    nextList = [-1 for i in range(len(p))]
    i = 0
    j = -1
    while i < len(p) - 1:
        if j == -1 or p[i] == p[j]:
            i += 1
            j += 1
            nextList[i] = j
        else:
            j = nextList[j]
    return nextList


def kmp(t, p):
    nextlist = getNextList(p)
    i = 0  # t 的下标
    j = 0  # p 的下标
    lent = len(t)
    lenp = len(p)
    while i < lent and j < lenp:
        if j == -1 or t[i] == p[j]:
            i += 1
            j += 1
        else:
            j = nextlist[j]
    if j == lenp:
        return i - j  # 匹配成功
    return -1  # 匹配失败


if __name__ == '__main__':
    t = 'bbc abcdabd abcdabcdabde'
    p = 'abcdabd'
    print(kmp(t, p))
    
输出：
4
```

# KMP找到所有匹配的子串的起始下标
找到待匹配串中满足模式匹配的所有子串的起始位置。

```python
# 获取 最长相同真前后缀长度数组
def getNextList(p):
    nextList = [-1 for i in range(len(p))]
    i = 0
    j = -1
    while i < len(p) - 1:
        if j == -1 or p[i] == p[j]:
            i += 1
            j += 1
            nextList[i] = j
        else:
            j = nextList[j]
    return nextList


def kmp(t, p):
    nextlist = getNextList(p)
    i = 0  # t 的下标
    j = 0  # p 的下标
    lent = len(t)
    lenp = len(p)
    res = []
    while i < lent:
        while j < lenp:
            # 防止i越界
            if i >= lent:
                break
            if j == -1 or t[i] == p[j]:
                i += 1
                j += 1
            else:
                j = nextlist[j]
        if j == lenp:
            res.append(i-j)
            j = 0
    return res


if __name__ == '__main__':
    t = 'bbc abcdabd abcdabcdabde'
    p = 'abcdabd'
    print(kmp(t, p))
    
输出：
[4, 16]
```



# 参考文献
[KMP算法（1）：如何理解KMP](https://segmentfault.com/a/1190000008575379)      
[[Algorithm] 字符串匹配算法——KMP算法](http://www.cnblogs.com/maybe2030/p/4633153.html)


