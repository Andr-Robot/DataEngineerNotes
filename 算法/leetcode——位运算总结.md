[toc]

# 准备知识
- **取反(NOT)**：使数字1成为0，0成为1。取反操作符用波浪线"`~`"表示
- **按位或(OR)**：两个相应的二进位中只要有一个为1，该位的结果值为1。按位或操作符是"`|`"
- **按位异或(XOR)**：两个数如果某位不同则该位为1，否则该位为0。按位异或运算符是"`^`"
- **按位与(AND)**：两个相应的二进位都为1，该位的结果值才为1，否则为0。按位与用'`&`'表示
- **逻辑移位**：应用逻辑移位时，移位后空缺的部分全部填0。符号有"`>>`"和"`<<`"

负数在计算机内部的存储方式是**补码**的形式。负数的补码就等于**其正数反码+1**。

# 136. 只出现一次的数字
[只出现一次的数字](https://leetcode-cn.com/problems/single-number/description/)
> 给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。


```
0010 0100
^
0010 0100
=
0000 0000
```

由以上可得知，相同数字做^异或运算，会得到0。   

# 137. 只出现一次的数字 II
[只出现一次的数字 II](https://leetcode-cn.com/problems/single-number-ii/description/)    
> 给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现了三次。找出那个只出现了一次的元素。

1. 对每个整数以32位表示，分别统计每一位上1的个数，最后该位数上的和对3取余数；
2. 如果余数不为0，则说明该位上只出现一次的元素，在该位上有1;
3. 通过移位操作和1做位与运算，则可求得当前元素在该位是否有1。

```python
class Solution(object):
    def singleNumber(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        # 参考https://shenjie1993.gitbooks.io/leetcode-python/137%20Single%20Number%20II.html
        res = 0
        # 将数值映射到32位的bit位上
        for i in range(32):
            sumone = 0
            tmp = 0
            for j in range(len(nums)):
                # 统计每个bit位上值为1出现的次数
                # x & 1 判断x的最后一位是否为1，为1则结果是1 不为1则结果是0
                # 每一轮统计要将数右移i位，去查看这个位上值是否为1
                sumone += (nums[j] >> i) & 1
            # sumone如果是3的倍数这说明这个位上出现了3次元素
            # 当余数不为0，则说明这个位上只出现了1次的数据在这个位置上有1
            # 然后将这个1左移i位就可以将只出现一次的那个数在这个位上取1
            # 按位或操作只要有一个为1，则结果为1
            tmp = sumone % 3
            # 处理负数的情况
            if i == 31 and tmp:
                res -= 1 << 31
            else:
                res |= tmp << i
        return res

if __name__ == '__main__':
    s = Solution()
    nums = [1, 1, 1, 2, 3, 3, 3]
    print(s.singleNumber(nums))
    
输出：
2
```

# 260. 只出现一次的数字 III
参见[只出现一次的数字 III](https://leetcode-cn.com/problems/single-number-iii/description/)   
> 给定一个整数数组 nums，其中恰好有两个元素只出现一次，其余所有元素均出现两次。 找出只出现一次的那两个元素。

```python
class Solution(object):
    def singleNumber(self, nums):
        """
        :type nums: List[int]
        :rtype: List[int]
        """
        # 参考https://blog.csdn.net/smile_watermelon/article/details/47750249
        # 先对数组中的每个数字进行异或运算，以为有相同的数字，
        # 故异或运算后会抵消，剩下的就是两个只出现一次的数字异或运算的结果
        # 假设两个出现次数为1的数分别为a b
        # z = a ^ b
        z = 0
        for i in nums:
            z ^= i
        # 找到z中第一个位1的二进制位
        # 因为z中二进制位1的位置说明a,b中一个位0 一个位1
        index = 0
        while z & 1 == 0:
            z >>= 1
            index += 1

        a = 0
        b = 0
        for i in nums:
            # 然后比较这个数字的二进制位在index这个位上是否为1，将数字分开
            if (i >> index) & 1:
                a ^= i
            else:
                b ^= i
        return [a,b]

if __name__ == '__main__':
    s = Solution()
    nums = [1,2,1,3,2,5]
    print(s.singleNumber(nums))

输出：
[3,5]
```



# 参考文献
[LeetCode——BitManipulation](https://github.com/xuelangZF/LeetCode/tree/master/BitManipulation)   
[位运算有什么奇技淫巧？](https://www.zhihu.com/question/38206659)    
[位运算简介及实用技巧（一）：基础篇](http://www.matrix67.com/blog/archives/263)    
[位运算简介及基本技巧](https://blog.yangx.site/2016/07/06/bit-operation-skills/)