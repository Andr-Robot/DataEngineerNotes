[toc]

一般回溯的问题有三种：
1. Find a path to success 有没有解
2. Find all paths to success 求所有解
    - 求所有解的个数
    - 求所有解的具体信息
3. Find the best path to success 求最优解


# 17. 电话号码的字母组合
[电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/description/) 
> 给定一个仅包含数字 2-9 的字符串，返回所有它能表示的字母组合。   
给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

参见[[Leetcode] Letter Combinations of a Phone Number 电话号码组合](https://segmentfault.com/a/1190000003766442)    

首先建一个表，来映射号码和字母的关系。然后对号码进行深度优先搜索，对于每一位，从表中找出数字对应的字母，这些字母就是本轮搜索的几种可能。

```python
class Solution:
    # 首先建一个表，来映射号码和字母的关系。然后对号码进行深度优先搜索，对于每一位，
    # 从表中找出数字对应的字母，这些字母就是本轮搜索的几种可能。
    def letterCombinations(self, digits):
        """
        :type digits: str
        :rtype: List[str]
        """
        if not digits:
            return []
        res = []
        self.helper(digits, res, [], 0)
        return res

    def helper(self, digits, res, tmp, index):
        # 建立映射表
        mlist = ['', '', 'abc', 'def', 'ghi', 'jkl', 'mno', 'pqrs', 'tuv', 'wxyz']
        if len(tmp) == len(digits):
            # 找到一种结果，加入列表中
            tmp = ''.join(tmp)
            res.append(tmp[:])
        else:
            # 找出当前位数字对应可能的字母
            curstr = mlist[int(digits[index])]
            # 对每个可能字母进行搜索
            for i in range(len(curstr)):
                tmp.append(curstr[i])
                self.helper(digits, res, tmp, index+1)
                tmp.pop()
```

# 22. 括号生成
[括号生成](https://leetcode-cn.com/problems/generate-parentheses/description/)
> 给出 n 代表生成括号的对数，请你写出一个函数，使其能够生成所有可能的并且有效的括号组合。

解析参见[[LeetCode] Generate Parentheses 生成括号](http://www.cnblogs.com/grandyang/p/4444160.html)     

这道题给定一个数字n，让生成共有n个括号的所有正确的形式，对于这种列出所有结果的题首先还是考虑用递归Recursion来解，由于字符串只有左括号和右括号两种字符，而且最终结果必定是左括号3个，右括号3个，所以我们定义两个变量left和right分别表示剩余左右括号的个数，如果在某次递归时，左括号的个数大于右括号的个数，说明此时生成的字符串中右括号的个数大于左括号的个数，即会出现')('这样的非法串，所以这种情况直接返回，不继续处理。如果left和right都为0，则说明此时生成的字符串已有3个左括号和3个右括号，且字符串合法，则存入结果中后返回。如果以上两种情况都不满足，若此时left大于0，则调用递归函数，注意参数的更新，若right大于0，则调用递归函数，同样要更新参数。


```python
class Solution:
    def generateParenthesis(self, n):
        """
        :type n: int
        :rtype: List[str]
        """
        res = []
        # 定义两个变量left和right分别表示剩余左右括号的个数
        left = n
        right = n
        self.helper(left, right, '', res)
        return res

    def helper(self, left, right, tmp, res):
        # 如果在某次递归时，左括号的个数大于右括号的个数，说明此时生成的字符串中右括号的个数大于左括号的个数，
        # 即会出现')('这样的非法串，所以这种情况直接返回，不继续处理。
        if left > right:
            return
        if left == 0 and right == 0:
            res.append(tmp)
        else:
            if left > 0:
                tmp = tmp + '('
                self.helper(left-1, right, tmp, res)
                tmp = tmp[:-1]
                # 可以使用下面这种方式，就不要考虑递归一层厚tmp需要将最后一位删掉
                # self.helper(left-1, right, tmp+'(', res)
            if right > 0:
                tmp = tmp + ')'
                self.helper(left, right-1, tmp, res)
                tmp = tmp[:-1]
                # self.helper(left, right-1, tmp+')', res)
```

# 77. 组合
[组合](https://leetcode-cn.com/problems/combinations/description/)    
> 给定两个整数 n 和 k，返回 1 ... n 中所有可能的 k 个数的组合。

```python
class Solution(object):
    def combine(self, n, k):
        """
        :type n: int
        :type k: int
        :rtype: List[List[int]]
        """
        # 回溯法
        # 先生成所有可能的数字
        nums = [i for i in range(1, n+1)]
        res = []
        self.helper(nums, res, [], 0, k)
        return res

    def helper(self, nums, res, tmp, index, limit):
        if len(tmp) > limit:
            return
        elif len(tmp) == limit:
            res.append(tmp[:])
            return
        else:
            for i in range(index, len(nums)):
                tmp.append(nums[i])
                self.helper(nums, res, tmp, i+1, limit)
                tmp.pop()
```



# 39. 组合总和
[组合总和](https://leetcode-cn.com/problems/combination-sum/description/)
> 给定一个无重复元素的数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。     
candidates 中的数字可以无限制重复被选取。

先将数组降序，然后使用回溯法去找到符合要求的组合。

```python
class Solution:
    def combinationSum(self, candidates, target):
        """
        :type candidates: List[int]
        :type target: int
        :rtype: List[List[int]]
        """
        # 先排序，降序
        nums = sorted(candidates, key=lambda x: x, reverse=True)
        res = []
        self.helper(nums, res, [], 0, 0, target)
        return res

    def helper(self, nums, res, tmp, index, cursum, target):
        if cursum > target:
            return
        elif cursum == target:
            res.append(tmp[:])
        else:
            for i in range(index, len(nums)):
                tmp.append(nums[i])
                cursum += nums[i]
                # 因为可以重复使用数字，所以迭代调用的时候可以继续从i开始
                self.helper(nums, res, tmp, i, cursum, target)
                tmp.pop()
                cursum -= nums[i]
```

# 40. 组合总和 II
[组合总和 II](https://leetcode-cn.com/problems/combination-sum-ii/description/)
> 给定一个数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。     
candidates 中的每个数字在每个组合中只能使用一次。

这道题的解法和上道题类似，只是要考虑candidates中可能有重复数字，且每个数只能用一次。所以就需要判断当遇到重复数字时候要跳过。

```python
class Solution:
    def combinationSum2(self, candidates, target):
        """
        :type candidates: List[int]
        :type target: int
        :rtype: List[List[int]]
        """
        # 先排序，降序
        nums = sorted(candidates, key=lambda x: x, reverse=True)
        res = []
        self.helper(nums, res, [], 0, 0, target)
        return res

    def helper(self, nums, res, tmp, index, cursum, target):
        if cursum > target:
            return
        elif cursum == target:
            res.append(tmp[:])
        else:
            for i in range(index, len(nums)):
                if i > index and nums[i] == nums[i-1]:
                    continue
                tmp.append(nums[i])
                cursum += nums[i]
                # 因为可以重复使用数字，所以迭代调用的时候可以继续从i开始
                self.helper(nums, res, tmp, i+1, cursum, target)
                tmp.pop()
                cursum -= nums[i]
```

# 216. 组合总和 III
[组合总和 III](https://leetcode-cn.com/problems/combination-sum-iii/description/)
> 找出所有相加之和为 n 的 k 个数的组合。组合中只允许含有 1 - 9 的正整数，并且每种组合中不存在重复的数字。    
输入: k = 3, n = 9   
输出: [[1,2,6], [1,3,5], [2,3,4]]


```python
class Solution:
    def combinationSum3(self, k, n):
        """
        :type k: int
        :type n: int
        :rtype: List[List[int]]
        """
        res = []
        # 组合中只允许含有 1 - 9 的正整数所以从9向1搜索
        self.helper(n, res, [], 9, 0, k)
        return res

    def helper(self, target, res, tmp, start, cursum, limit):
        if cursum == target and len(tmp) == limit:
            res.append(tmp[:])
        else:
            for i in range(start, 0, -1):
                tmp.append(i)
                cursum += i
                self.helper(target, res, tmp, i-1, cursum, limit)
                tmp.pop()
                cursum -= i
```

# 131. 分割回文串
[分割回文串](https://leetcode-cn.com/problems/palindrome-partitioning/description/)
> 给定一个字符串 s，将 s 分割成一些子串，使每个子串都是回文串。    
返回 s 所有可能的分割方案。

详解参见[Leetcode 131. 分割回文串](http://jackniulin.com/2018/05/30/leetcode131/)    
回溯法，还需要一个判断子串是否是回文串的函数。

```python
class Solution:
    def partition(self, s):
        """
        :type s: str
        :rtype: List[List[str]]
        """
        res = []
        self.helper(s, res, [], 0)
        return res

    def helper(self, s, res, tmp, index):
        if index == len(s):
            res.append(tmp[:])
        else:
            for i in range(index, len(s)):
                # 判断当前子串是否为回文串，如果是回文串则继续将剩下的子串分割成回文串
                if self.ispalindromic(s, index, i):
                    tmp.append(s[index:i+1])
                    self.helper(s, res, tmp, i+1)
                    tmp.pop()

    def ispalindromic(self, s, start, end):
        while start < end:
            if s[start] != s[end]:
                return False
            start += 1
            end -= 1
        return True
```


# 参考文献
[算法基础总结](http://www.linguang.space/article.php?type=all_article&id=14977788276880)     
[Backtracking回溯法(又称DFS,递归)全解](https://segmentfault.com/a/1190000006121957)