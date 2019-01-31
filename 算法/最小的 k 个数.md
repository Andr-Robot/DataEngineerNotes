> 例如输入 4 、5 、1、6、2、7、3 、8 这 8 个数字，则最小的 4 个数字是 1 、2、3 、4

# 排序
先把数组按升序/降序进行排序，然后输出 K 个最小/最大的数。

# 部分排序
`$O(n*k)$`    
由于我们只需要找出最小/最大的 k 个数，所以我们可以进行部分排序，比如简单选择排序 和 冒泡排序，它们每一趟都能把一个最小/最大元素放在最终位置上，所以进行 k 趟就能把 n 个数中的前 k 个排序出来。

```python
def select_sort(nums, n, k):
    for i in range(k):
        mmin = i
        for j in range(i + 1, n):
            if nums[j] < nums[mmin]:
                mmin = j
        if mmin != i:
            tmp = nums[mmin]
            nums[mmin] = nums[i]
            nums[i] = tmp

    return nums

def bubble_sort(nums, n, k):
    for i in range(k):
        flag = False
        # for j in range(n-1, i, -1):
        #     if nums[j-1] > nums[j]:
        #         tmp = nums[j-1]
        #         nums[j-1] = nums[j]
        #         nums[j] = tmp
        #         flag = True
        for j in range(i+1, n):
            if nums[j] < nums[j-1]:
                tmp = nums[j-1]
                nums[j-1] = nums[j]
                nums[j] = tmp
                flag = True
        if flag:
            break
    return nums


if __name__ == '__main__':
    mlist = [10, 40, 60, 10, 30, 20, 50]
    k = 4
    res = select_sort(mlist, len(mlist), k)
    print(res)
    print(res[:k])
    res1 = bubble_sort(mlist, len(mlist), k)
    print(res1)
    print(res1[:k])

输出：
[10, 10, 20, 30, 40, 60, 50]
[10, 10, 20, 30]
[10, 10, 20, 30, 40, 50, 60]
[10, 10, 20, 30]
```



# 快排划分
` $O(n*\log_2 k)$`     
如果不考虑输出是否有序可以使用如下方法解决：   
类似于快速选择算法的思想，通过找到i == k的那个位置，在i右面的都是小于mlist[i]的数，最后返回mlist[:k]。

```python
def helper(mlist, start, end):
    if start == end:
        return start
    if start < end:
        i = start
        j = end
        key = mlist[start]
        while i < j:
            while i < j and mlist[j] >= key:
                j -= 1
            if i < j:
                mlist[i] = mlist[j]
                i += 1
            while i < j and mlist[i] <= key:
                i += 1
            if i < j:
                mlist[j] = mlist[i]
                j -= 1
        mlist[i] = key
        return i


if __name__ == '__main__':
    mlist = [10, 40, 60, 10, 30, 20, 50]
    start = 0
    end = len(mlist) - 1
    k = 4
    index = helper(mlist, 0, len(mlist) - 1)
    while index != k:
        if index < k:
            start = index + 1
        else:
            end = index - 1
        index = helper(mlist, start, end)
    print(mlist[:k])
    
输出：
[10, 20, 30, 10]
```

# 最大堆
`$O(n * \log_2 k)$`    
第二种方法就是使用堆排序，初始化一个容量为k的最大堆，然后从k+1向后遍历数组，并更新最小堆。

```python
def max_percDown(data, i, n):
    l = 2 * i + 1
    while l <= n:
        if l < n and data[l] < data[l + 1]:
            l += 1
        if data[i] < data[l]:
            tmp = data[l]
            data[l] = data[i]
            data[i] = tmp
            i = l
            l = 2 * i + 1
        else:
            break

def buildMaxHeap(nums, n):
    i = n // 2 - 1
    while i >= 0:
        max_percDown(nums, i, n-1)
        i -= 1

def topK(nums, n, k):
    # 构建最大堆
    buildMaxHeap(nums, k)
    # 将剩下的数字依次和最大堆的堆顶元素值对比
    # 如果小于堆顶元素，则替换堆顶元素值
    # 最后进行下沉操作重新构建最大堆
    for i in range(k, n):
        if nums[i] < nums[0]:
            tmp = nums[0]
            nums[0] = nums[i]
            nums[i] = tmp
            max_percDown(nums, 0, k-1)

if __name__ == '__main__':
    mlist = [10, 40, 60, 10, 30, 20, 50]
    k = 4
    topK(mlist, len(mlist), k)
    print(mlist[:k])

输出：
[30, 20, 10, 10]
```

# 参考文献
[华为OJ2051-最小的K个数（Top K问题）](https://songlee24.github.io/2015/03/21/hua-wei-OJ2051/)