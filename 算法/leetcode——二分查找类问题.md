[toc]

# 744. 寻找比目标字母大的最小字母
[744. 寻找比目标字母大的最小字母](https://leetcode-cn.com/problems/find-smallest-letter-greater-than-target/description/)    
> 给定一个只包含小写字母的有序数组letters 和一个目标字母 target，寻找有序数组里面比目标字母大的最小字母。    
数组里字母的顺序是循环的。举个例子，如果目标字母target = 'z' 并且有序数组为 letters = ['a', 'b']，则答案返回 'a'。

```python
class Solution:
    def nextGreatestLetter(self, letters, target):
        """
        :type letters: List[str]
        :type target: str
        :rtype: str
        """
        left = 0
        right = len(letters) - 1
        while left <= right:
            mid = (left + right) // 2
            if ord(letters[mid]) <= ord(target):
                if ord(target) >= ord(letters[right]):
                    return letters[left]
                else:
                    left = mid + 1
            elif ord(letters[mid]) > ord(target):
                if ord(target) < ord(letters[left]):
                    return letters[left]
                else:
                    right = mid
```

# 69. x 的平方根
[69. x 的平方根](https://leetcode-cn.com/problems/sqrtx/description/)
> 实现 int sqrt(int x) 函数。    
计算并返回 x 的平方根，其中 x 是非负整数。    
由于返回类型是整数，结果只保留整数的部分，小数部分将被舍去。

```
class Solution:
    def mySqrt(self, x):
        """
        :type x: int
        :rtype: int
        """
        if x <= 1:
            return x
        left = 0
        right = x
        while left < right:
            mid = (left + right) // 2
            res = mid * mid
            if res == x:
                return mid
            elif res > x:
                right = mid
            elif res < x:
                left = mid + 1
        return left - 1
```

# 441. 排列硬币
[441. 排列硬币](https://leetcode-cn.com/problems/arranging-coins/description/)
> 你总共有 n 枚硬币，你需要将它们摆成一个阶梯形状，第 k 行就必须正好有 k 枚硬币。    
给定一个数字 n，找出可形成完整阶梯行的总行数。    
n 是一个非负整数，并且在32位有符号整型的范围内。

```python
class Solution:
    def arrangeCoins(self, n):
        """
        :type n: int
        :rtype: int
        """
        left = 0
        right = n
        while left <= right:
            mid = (left + right) // 2
            sum = mid * (mid + 1) // 2
            if sum == n:
                return mid
            elif sum < n:
                left = mid + 1
            elif sum > n:
                right = mid - 1
        return left - 1
```


# 852. 山脉数组的峰顶索引
[852. 山脉数组的峰顶索引](https://leetcode-cn.com/problems/peak-index-in-a-mountain-array/description/)    
> 我们把符合下列属性的数组 A 称作山脉：    
`A.length >= 3`    
存在 `0 < i < A.length - 1` 使得`A[0] < A[1] < ... A[i-1] < A[i] > A[i+1] > ... > A[A.length - 1]`     
给定一个确定为山脉的数组，返回任何满足 `A[0] < A[1] < ... A[i-1] < A[i] > A[i+1] > ... > A[A.length - 1]` 的 `i` 的值。

**思路：利用二分，根绝mid处的数值和其左右两边的数值的比较，确定搜索的范围。**

```python
class Solution:
    def peakIndexInMountainArray(self, A):
        """
        :type A: List[int]
        :rtype: int
        """
        left = 0
        right = len(A) - 1
        while left <= right:
            mid = (left + right) // 2
            if A[mid - 1] < A[mid] and A[mid] > A[mid + 1]:
                return mid
            elif A[mid - 1] < A[mid] < A[mid + 1]:
                left = mid + 1
            elif A[mid + 1] < A[mid] < A[mid - 1]:
                right = mid - 1
```

# 162. 寻找峰值
[162. 寻找峰值](https://leetcode-cn.com/problems/find-peak-element/description/)
> 峰值元素是指其值大于左右相邻值的元素。     
给定一个输入数组 nums，其中 nums[i] ≠ nums[i+1]，找到峰值元素并返回其索引。    
数组可能包含多个峰值，在这种情况下，返回任何一个峰值所在位置即可。    
你可以假设 nums[-1] = nums[n] = -∞。

- 规律一：如果nums[i] > nums[i+1]，则在i之前一定存在峰值元素
- 规律二：如果nums[i] < nums[i+1]，则在i+1之后一定存在峰值元素

```python
class Solution:
    def findPeakElement(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        left = 0
        right = len(nums) - 1
        while left < right:
            mid = (left + right) // 2
            if nums[mid] > nums[mid + 1]:
                right = mid
            else:
                left = mid + 1
        return left
```



# 153. 寻找旋转排序数组中的最小值
[153. 寻找旋转排序数组中的最小值](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/description/)
> 假设按照升序排序的数组在预先未知的某个点上进行了旋转。
( 例如，数组 [0,1,2,4,5,6,7] 可能变为 [4,5,6,7,0,1,2] )。    
请找出其中最小的元素。    
你可以假设数组中不存在重复元素。


如果这个升序排序的有序数组确实按照某个未知的位置旋转，只有两种可能：
1. a [mid]  > a [left] && a [mid]> a [right]，所以，mid位于较大的那部分，较小的部分位于mid右边，应该向右走；
2. a [mid] <a [left] && a [mid] <a [right]，所以，mid位于较小的那部分，为了找到最小的元素，应该向左走；

如果这个数组没有旋转(或者是循环旋转完)，只需要往左走，a[mid]总是< a[right]；

结论：mid > right，向右走；mid < right，向左走；

```python
class Solution:
    def findMin(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        left = 0
        right = len(nums) - 1
        while left < right:
            mid = (left + right) // 2
            if nums[mid] > nums[right]:
                # 因为当nums[mid] > nums[right]时，则可以确定的是num[mid]和左边数值是连续递增的
                # 所以寻找最小值，应该以它的下一个为起点去搜索最小值
                left = mid + 1  
            else:
                right = mid
        if nums[left] > nums[right]:
            return nums[right]
        else:
            return nums[left]
```

# 33. 搜索旋转排序数组
[33. 搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/description/)    
> 假设按照升序排序的数组在预先未知的某个点上进行了旋转。     
( 例如，数组 [0,1,2,4,5,6,7] 可能变为 [4,5,6,7,0,1,2] )。    
搜索一个给定的目标值，如果数组中存在这个目标值，则返回它的索引，否则返回 -1 。   
你可以假设数组中不存在重复的元素。   
你的算法时间复杂度必须是 O(log n) 级别。

**思路**和`153. 寻找旋转排序数组中的最小值`中的思路类似，利用二分的方法，然后比较中间那个数和最右端（最左端）数的大小，最后在去确定目标数值所在的范围继续进行二分查找。

```python
class Solution:
    def search(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        left = 0
        right = len(nums) - 1
        while left <= right:
            mid = (left + right) // 2
            if nums[mid] == target:
                return mid
            if nums[mid] <= nums[right]:
                if nums[mid] < target <= nums[right]:
                    left = mid + 1
                else:
                    right = mid - 1
            else:
                if nums[left] <= target < nums[mid]:
                    right = mid - 1
                else:
                    left = mid + 1
        return -1
```

# 81. 搜索旋转排序数组 II
[81. 搜索旋转排序数组 II](https://leetcode-cn.com/problems/search-in-rotated-sorted-array-ii/description/)
> 假设按照升序排序的数组在预先未知的某个点上进行了旋转。    
( 例如，数组 [0,0,1,2,2,5,6] 可能变为 [2,5,6,0,0,1,2] )。     
编写一个函数来判断给定的目标值是否存在于数组中。若存在返回 true，否则返回 false。

**思路，就是利用mid left right三个位置的数值的大小，然后确定下次寻找的上下界。**

```python
class Solution:
    def search(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: bool
        """
        left = 0
        right = len(nums) - 1
        while left <= right:
            mid = (left + right) // 2
            if nums[mid] == target:
                return True
            if nums[left] == nums[mid] and nums[right] == nums[mid]:
                for i in range(left, right):
                    if nums[i] == target:
                        return True
                return False
            if nums[mid] < target:
                if nums[right] >= target or nums[mid] >= nums[left]:
                    left = mid + 1
                else:
                    right = mid - 1
            else:
                if nums[left] <= target or nums[mid] <= nums[right]:
                    right = mid - 1
                else:
                    left = mid + 1
        return False
```




