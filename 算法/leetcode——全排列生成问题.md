[toc]

# 784. 字母大小写全排列
[字母大小写全排列](https://leetcode-cn.com/problems/letter-case-permutation/description/)

> 给定一个字符串S，通过将字符串S中的每个字母转变大小写，我们可以获得一个新的字符串。返回所有可能得到的字符串集合。

参看[Leetcode学习笔记-784-字母大小写全排列](https://unclegem.cn/2018/05/13/Leetcode%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0-784-%E5%AD%97%E6%AF%8D%E5%A4%A7%E5%B0%8F%E5%86%99%E5%85%A8%E6%8E%92%E5%88%97/)

```java
class Solution {
    public List<String> letterCasePermutation(String S) {
		List<String> res = new LinkedList<>();
		dfs("", S, res, 0);
		return res;
	}

	public void dfs(String pre, String S, List<String> res, int index) {
		if (index == S.length()) {
			res.add(pre);
		} else {
			char ch = S.charAt(index);
			if (!Character.isLetter(ch)) {
				dfs(pre + ch, S, res, index + 1);
			}else {
				ch = Character.toLowerCase(ch);
				dfs(pre+ch, S, res, index+1);
				ch = Character.toUpperCase(ch);
				dfs(pre+ch, S, res, index+1);
			}
		}
	}
}
```


```python
class Solution(object):
    # 回溯法:当前字符若是数字则直接搜索下一个位置，若当前是英文字母，则分成两种情况分别向下搜索。
    # 当当前下标与字符串长度相等则将变量串添加到结果集合中。
    def letterCasePermutation(self, S):
        """
        :type S: str
        :rtype: List[str]
        """
        res = []
        self.helper("", S, res, 0)
        return res

    def helper(self, pre, S, res, index):
        if index == len(S):
            res.append(pre)
        else:
            ch = S[index]
            if str.isdigit(ch):
                self.helper(pre+ch, S, res, index+1)
            else:
                ch = str.lower(ch)
                self.helper(pre+ch, S, res, index+1)
                ch = str.upper(ch)
                self.helper(pre+ch, S, res, index+1)
```


# 46. 全排列
[全排列](https://leetcode-cn.com/problems/permutations/description/)   
> 给定一个没有重复数字的序列，返回其所有可能的全排列。

解法参看[[算法教程] 全排列——视频](https://www.bilibili.com/video/av9830088?from=search&seid=1804004168200076381)   


```python
class Solution:
    def permute(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        res = []
        # 采用前后元素交换的办法，dfs解题
        # 就是第一个数分别以后面的数进行交换
        self.dfs(nums, 0, res)
        return res

    def dfs(self, nums, begin, res):
        if begin == len(nums):
            res.append(nums[:])
            return
        else:
            # 第i个数分别与它后面的数字交换就能得到新的排列
            for i in range(begin, len(nums)):
                self.swap(nums, begin, i)
                self.dfs(nums, begin+1, res)
                # 将前面的调换恢复回去
                self.swap(nums, begin, i)

    def swap(self, nums, i, j):
        temp = nums[i]
        nums[i] = nums[j]
        nums[j] = temp

s = Solution()
nums = [1,3,4]
r = s.permute(nums)
print(r)

输出：
[[1, 3, 4], [1, 4, 3], [3, 1, 4], [3, 4, 1], [4, 3, 1], [4, 1, 3]]
```

# 31. 下一个排列
[下一个排列](https://leetcode-cn.com/problems/next-permutation/description/) 
> 实现获取下一个排列的函数，算法需要将给定数字序列重新排列成字典序中下一个更大的排列。    
如果不存在下一个更大的排列，则将数字重新排列成最小的排列（即升序排列）。

## 交换法
参见[[LeetCode] Next Permutation 下一个排列](http://www.cnblogs.com/grandyang/p/4428207.html)    
> 这道题让我们求下一个排列顺序，有题目中给的例子可以看出来，如果给定数组是降序，则说明是全排列的最后一种情况，则下一个排列就是最初始情况，如下的一个数组:   
1　　2　　7　　4　　3　　1   
下一个排列为：   
1　　3　　1　　2　　4　　7   
如果从末尾往前看，数字逐渐变大，到了2时才减小的，然后我们再从后往前找第一个比2大的数字，是3，那么我们交换2和3，再把此时3后面的所有数字转置一下即可，步骤如下：   
1　　**2**　　7　　4　　3　　1   
1　　**2**　　7　　4　　**3**　　1   
1　　**3**　　7　　4　　**2**　　1   
1　　3　　**1**　　**2**　　**4**　　**7**   


```python
class Solution:
    def nextPermutation(self, nums):
        """
        :type nums: List[int]
        :rtype: void Do not return anything, modify nums in-place instead.
        """
        # 从后往前找到序列中第一个非递增的数的索引
        i = len(nums) - 2
        while i >= 0:
            if nums[i] < nums[i+1]:
                break
            i -= 1
        if i < 0:
            nums.sort()
            return
        # 从后往前找到后面序列中第一个比nums[i]大的数
        j = len(nums) - 1
        while j > i:
            if nums[j] > nums[i]:
                break
            j -= 1
        # 交换位置
        self.swap(nums, i, j)
        # 将i后面的序列逆序
        nums[i+1:] = nums[i+1:][::-1]

    def swap(self, nums, i, j):
        temp = nums[i]
        nums[i] = nums[j]
        nums[j] = temp

s = Solution()
nums = [1,2]
r = s.nextPermutation(nums)
print(nums)

输出：
[2, 1]
```

## 深度优先搜索
参看[[Leetcode] Permutations 全排列——深度优先搜索法](https://segmentfault.com/a/1190000003765975#articleHeader5)

我们还可以简单的使用深度优先搜索来解决这题。每一轮搜索选择一个数加入列表中，同时我们还要维护一个全局的布尔数组，来标记哪些元素已经被加入列表了，这样在下一轮搜索中要跳过这些元素。     

```python
class Solution:
    # 每一轮搜索选择一个数加入列表中，同时我们还要维护一个全局的布尔数组，
    # 来标记哪些元素已经被加入列表了，这样在下一轮搜索中要跳过这些元素。
    def permute(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        # 新建一个used数组，大小与nums相同，用来标记在本次DFS读取中，位置i的元素是否已经被添加到list中了。
        used = [False for i in range(len(nums))]
        res = []
        tmp = []  # 临时数组用于存放每一轮搜索访问到的元素
        self.helper(nums, tmp, res, used)
        return res

    def helper(self, nums, tmp, res, used):
        if len(tmp) == len(nums):
            res.append(tmp[:])
        else:
            i = 0
            while i < len(nums):
                # 遇到已经加过的元素就跳过
                if used[i]:
                    i += 1
                    continue
                # 加入该元素后继续搜索
                used[i] = True
                tmp.append(nums[i])
                self.helper(nums, tmp, res, used)
                tmp.pop(-1)
                used[i] = False
                i += 1
```


```java
class Solution {
	boolean[] used;
	List<List<Integer>> res;

	public List<List<Integer>> permute(int[] nums) {
		res = new LinkedList<List<Integer>>();
		used = new boolean[nums.length];
		List<Integer> tmp = new LinkedList<Integer>();
		helper(nums, tmp);
		return res;
	}

	private void helper(int[] nums, List<Integer> tmp) {
		if (tmp.size() == nums.length) {
			List<Integer> list = new LinkedList<Integer>(tmp);
			res.add(list);
		} else {
			for (int idx = 0; idx < nums.length; idx++) {
				// 遇到已经加过的元素就跳过
				if (used[idx]) {
					continue;
				}
				// 加入该元素后继续搜索
				used[idx] = true;
				tmp.add(nums[idx]);
				helper(nums, tmp);
				tmp.remove(tmp.size() - 1);
				used[idx] = false;
			}
		}
	}
}
```


# 47. 全排列 II
[全排列 II](https://leetcode-cn.com/problems/permutations-ii/description/)

> 给定一个可包含重复数字的序列，返回所有不重复的全排列。

解析见[[LeetCode] Permutations II 全排列之二](http://www.cnblogs.com/grandyang/p/4359825.html)   

这题和上题的深度优先搜索很相似，区别在于：
1. 要先将数组排序，确保重复的元素是在一起的。
2. 除了不能加入之前出现过的元素之外，还不能加入本轮搜索中出现过的元素。如何判断哪些元素本轮出现过呢？我们加过一个数字并搜索后，在这一轮中把剩余的重复数字都跳过就行了，保证每一轮只有一个unique的数字出现。这和Combination Sum II中跳过重复元素的方法是一样的，注意要判断nums[i] == nums[i + 1]，因为for循环结束时i还会额外加1，我们要把i留在最后一个重复元素处。

```python
class Solution:
    # 每一轮搜索选择一个数加入列表中，同时我们还要维护一个全局的布尔数组，
    # 来标记哪些元素已经被加入列表了，这样在下一轮搜索中要跳过这些元素。
    def permuteUnique(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        # 先对数组排序，使得大小相同的元素排在一起。
        nums.sort()
        # 新建一个used数组，大小与nums相同，用来标记在本次DFS读取中，位置i的元素是否已经被添加到list中了。
        used = [False for i in range(len(nums))]
        res = []
        tmp = []  # 临时数组用于存放每一轮搜索访问到的元素
        self.helper(nums, tmp, res, used)
        return res

    def helper(self, nums, tmp, res, used):
        if len(tmp) == len(nums):
            res.append(tmp[:])
        else:
            i = 0
            while i < len(nums):
                # 遇到已经加过的元素就跳过
                if used[i]:
                    i += 1
                    continue
                # 加入该元素后继续搜索
                used[i] = True
                tmp.append(nums[i])
                self.helper(nums, tmp, res, used)
                tmp.pop(-1)
                used[i] = False
                # 跳过本轮的重复元素，确保每一轮只会加唯一的数字
                while i < len(nums)-1 and nums[i] == nums[i+1]:
                    i += 1
                i += 1
```


```java
class Solution {
	boolean[] used;
	List<List<Integer>> res;

	public List<List<Integer>> permuteUnique(int[] nums) {
		res = new LinkedList<List<Integer>>();
		used = new boolean[nums.length];
		Arrays.sort(nums);
		List<Integer> tmp = new LinkedList<Integer>();
		helper(nums, tmp);
		return res;
	}

	private void helper(int[] nums, List<Integer> tmp) {
		if (tmp.size() == nums.length) {
			List<Integer> list = new LinkedList<Integer>(tmp);
			res.add(list);
		} else {
			for (int idx = 0; idx < nums.length; idx++) {
				// 遇到已经加过的元素就跳过
				if (used[idx]) {
					continue;
				}
				// 加入该元素后继续搜索
				used[idx] = true;
				tmp.add(nums[idx]);
				helper(nums, tmp);
				tmp.remove(tmp.size() - 1);
				used[idx] = false;
				// 跳过本轮的重复元素，确保每一轮只会加unique的数字
				while (idx < nums.length - 1 && nums[idx] == nums[idx + 1]) {
					idx++;
				}
			}
		}
	}
}
```


# 参考文献
[一次搞懂全排列——LeetCode四道Permutations问题详解](https://blog.csdn.net/Jacky_chenjp/article/details/66477538)    
[leetcode 全排列详解](https://www.hrwhisper.me/leetcode-permutations/)   
[[Leetcode] Permutations 全排列](https://segmentfault.com/a/1190000003765975)


