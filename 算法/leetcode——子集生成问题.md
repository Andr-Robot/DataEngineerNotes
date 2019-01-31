[toc]

# 78. 子集
[子集](https://leetcode-cn.com/problems/subsets/description/)

> 给定一组不含重复元素的整数数组 nums，返回该数组所有可能的子集（幂集）。

参见[[Leetcode] Subset 子集](https://segmentfault.com/a/1190000003498803)

## 回溯法
这个题是回溯法最经典的题，因为要找到所有的子集，所以不需要任何判断条件就把路径加入到结果中即可。

```python
class Solution:
    def subsets(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        # 使用回溯法
        res = []
        self.helper(nums, res, [], 0)
        return res

    def helper(self, nums, res, tmp, idx):
        res.append(tmp[:])
        for i in range(idx, len(nums)):
            tmp.append(nums[i])
            self.helper(nums, res, tmp, i+1)
            tmp.pop()
```

## 位运算法
求子集问题就是求组合问题。数组中的n个数可以用n个二进制位表示，当某一位为1表示选择对应的数，为0表示不选择对应的数。

```python
class Solution:
    def subsets(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        n = len(nums)
        # n个数的子集有2^n个
        m = 1 << n
        res = []
        for i in range(m):
            tmp = []
            for j in range(n):
                # 1左移j位，监测i的哪一位为1，为1的话添加到列表，按位与：两个相应的二进位都为1，该位的结果值才为1
                flag = i & (1 << j)
                if flag:
                    tmp.append(nums[j])
            res.append(tmp)
        return res
```

# 90. 子集 II
[子集 II](https://leetcode-cn.com/problems/subsets-ii/description/)    
> 给定一个可能包含重复元素的整数数组 nums，返回该数组所有可能的子集（幂集）。

思路和上题一样，区别在于如果有重复的只能加一次。好在我们已经先将数组排序（数组中有重复一般都可以用排序解决），所以重复元素是相邻的，我们为了保证重复元素只加一次，要把这些重复的元素在同一段逻辑中一起处理，而不是在递归中处理，不然就太麻烦了。所以我们可以先统计好重复的有n个，然后分别在集合中加上0个，1个，2个...n个，然后再分别递归。

```python
class Solution:
    def subsetsWithDup(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        nums.sort()
        res = []
        self.helper(nums, res, [], 0)
        return res

    def helper(self, nums, res, tmp, index):
        res.append(tmp[:])
        for i in range(index, len(nums)):
            if i > index and nums[i] == nums[i-1]:
                continue
            tmp.append(nums[i])
            self.helper(nums, res, tmp, i+1)
            tmp.pop()
```

# 参考文献
[【LeetCode】78. Subsets 解题报告](https://blog.csdn.net/fuxuemingzhu/article/details/79359540)    
[[Leetcode] Subset 子集](https://segmentfault.com/a/1190000003498803)