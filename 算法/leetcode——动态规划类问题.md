* [背包问题](#背包问题)
    * [0-1背包问题](#0-1背包问题)
        * [不考虑价值的背包问题](#不考虑价值的背包问题)
        * [考虑物品价值的背包问题](#考虑物品价值的背包问题)
        * [考虑物品价值的背包问题（输出最优装入方案）](#考虑物品价值的背包问题输出最优装入方案)
    * [完全背包](#完全背包)
    * [多重背包](#多重背包)
* [凑硬币](#凑硬币)
* [最长公共子序列和最长公共子串问题（LCS）](#最长公共子序列和最长公共子串问题lcs)
    * [最长公共子序列](#最长公共子序列)
    * [最长公共子串](#最长公共子串)
* [最长递增子序列](#最长递增子序列)
* [最长回文子串](#最长回文子串)
* [最长回文子序列](#最长回文子序列)
* [377. 组合总和 Ⅳ](#377-组合总和-ⅳ)
* [132. 分割回文串 II](#132-分割回文串-ii)
* [参考文献](#参考文献)

解决动态规划类问题，分为两步：
1. 确定状态；
2. 根据状态列状态转移方程。

# 背包问题
## 0-1背包问题
> 有`N`件物品和一个容量为`V`的背包， **每种物品均只有一件**。第`i`件物品的费用是`c[i]`，价值是`w[i]`。求解将哪些物品装入背包可使价值总和最大。 

### 不考虑价值的背包问题
> 在`n`个物品中挑选若干物品装入背包，最多能装多满？假设背包的大小为`m`，每个物品的大小为`A[i]`

这个题可以理解为选取若干物品放入背包，使背包剩余空间最小。   
**解题思路**：   
有一个容量为`m`的背包，有`n`个物品，尽量多装物品，使背包尽量的重。   
有背包和物品，物品有**放和不放**两种状态，放置的时候可能会对应各种容量，当前的容量下可以放置进的最多的物品取决于**上一个物品放置时在能够达到的最大状态**和**当前物品放入时能够达到的最大状态**，然后取最大值。    
状态为`sumres[i][j]`：表示将第`i`个物品放入容量为`j`背包后背包当时的最大体积值。   
状态转移方程为：

<img src="https://latex.codecogs.com/gif.latex?sumres[i][j]=\begin{cases}max(sumres[i-1][j],&space;sumres[i-1][j-nums[i]]&space;&plus;&space;nums[i])&j>=nums[i]\\\\&space;sumres[i-1][j]&j<nums[i]\end{cases}" title="sumres[i][j]=\begin{cases}max(sumres[i-1][j], sumres[i-1][j-nums[i]] + nums[i])&j>=nums[i]\\\\ sumres[i-1][j]&j<nums[i]\end{cases}" />        

其中`nums[i]`代表第`i`个物品的体积。   

例如`n=5`，则`nums = [5, 4, 7, 2, 6]`，将这些物品放入背包中，如下所示：

背包容量 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 |
---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---
nums[0] | 0 | 0 | 0 | 0 | 0 | 0 | 5 | 5 | 5 | 5 | 5 | 5 | 5 | 5 | 5 |
nums[1] | 0 | 0 | 0 | 0 | 4 | 5 | 5 | 5 | 5 | 9 | 9 | 9 | 9 | 9 | 9 |
nums[2] | 0 | 0 | 0 | 0 | 4 | 5 | 5 | 7 | 7 | 9 | 9 | 11 | 12 | 12 | 12 |
nums[3] | 0 | 0 | 2 | 2 | 4 | 5 | 6 | 7 | 7 | 9 | 9 | 11 | 12 | 13 | 14 |
nums[4] | 0 | 0 | 2 | 2 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 |

**代码实现**：

```python
class Solution:
    def backPack(self, m, nums):
        if nums is None or m == 0:
            return 0
        # 初始化一个二维数组用于存储 背包在当前状态下最大的存储体积
        sumres = [[0 for i in range(m+1)] for j in range(len(nums))]
        # 当背包容量是0，则背包能放入物品的最大体积也就是0，所以sumres[i][0] = 0
        # 当背包容量为j，则当j>=nums[0]时，才可以将第一个物品放入
        for j in range(m+1):
            if j >= nums[0]:
                sumres[0][j] = nums[0]
        for i in range(1, len(nums)):
            for j in range(1, m+1):
                if j >= nums[i]:
                    # 此时考虑是否将nums[i]放入背包中，取是否放入的最大值
                    sumres[i][j] = max(sumres[i-1][j], sumres[i-1][j-nums[i]] + nums[i])
                else:
                    sumres[i][j] = sumres[i-1][j]
        return sumres[len(nums)-1][m]
        
s = Solution()
nums = [5, 4, 7, 2, 6]
print(s.backPack(14, nums))

输出：
14
```

使用O(m)的空间复杂度：

```
def helper(nums, m, n):
    # 初始化一维数组
    dp = [0 for i in range(m + 1)]
    # 注意，这里的起始值是 0
    for i in range(n):
        # 注意，这里是用逆序的方式遍历
        for j in range(m, nums[i] - 1, -1):
            dp[j] = max(dp[j], dp[j - nums[i]] + nums[i])
    return dp[-1]

if __name__ == '__main__':
    m = 14  # 容量为m的背包
    n = 5  # 物品个数
    nums = [5, 4, 7, 2, 6]  # 物品重量
    helper(nums, m, n)
```


### 考虑物品价值的背包问题
> 给出`n`个物品的体积`A[i]`和其价值`V[i]`，将他们装入一个大小为`m`的背包，最多能装入的总价值有多大？

考虑到价值问题，状态不发生变化，只是对于状态我们所记录的内容方式变化，我们现在记录的是其价值，而不是其放置的物品的大小。在记录价值的时我们需要考虑前提条件是该物品的体积要不大于当前背包的容量。    

用**子问题定义状态**：即`f[i][v]`表示前`i`件物品恰放入一个容量为`v`的背包可以获得的最大价值。则其状态转移方程便是：

<img src="https://latex.codecogs.com/gif.latex?f[i][v]=\begin{cases}max(f[i-1][v],&space;f[i-1][v-nums[i]]&space;&plus;&space;value[i])&v>=nums[i]\\\\&space;f[i-1][v]&v<nums[i]\end{cases}" title="f[i][v]=\begin{cases}max(f[i-1][v], f[i-1][v-nums[i]] + value[i])&v>=nums[i]\\\\ f[i-1][v]&v<nums[i]\end{cases}" />        

其中`nums[i]`代表第`i`个物品的体积。    

把这个过程理解下，在前`i`件物品放进容量`v`的背包时，它有两种情况:    
- 情况一: 第`i`件不放进去，这时所得价值为:`f[i-1][v]`
- 情况二: 第`i`件放进去，这时所得价值为：`f[i-1][v-nums[i]] + value[i]` 

例如`n=5`，则`nums = [5, 4, 7, 2, 6]`，价值为`value=[12,3,10,3,6]`。将这些物品放入背包中，如下所示：   
背包容量 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 |
---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---
nums[0] | 0 | 0 | 0 | 0 | 0 | 12 | 12 | 12 | 12 | 12 | 12 | 12 | 12 | 12 | 12 |
nums[1] | 0 | 0 | 0 | 0 | 3 | 12 | 12 | 12 | 12 | 15 | 15 | 15 | 15 | 15 | 15 |
nums[2] | 0 | 0 | 0 | 0 | 3 | 12 | 12 | 12 | 12 | 15 | 15 | 15 | 22 | 22 | 22 |
nums[3] | 0 | 0 | 3 | 3 | 3 | 12 | 12 | 15 | 15 | 15 | 15 | 18 | 22 | 22 | 25 |
nums[4] | 0 | 0 | 3 | 3 | 3 | 12 | 12 | 15 | 15 | 15 | 15 | 18 | 22 | 22 | 25 |

**代码实现**：

```python   
class Solution:
    def backPack(self, m, nums, value):
        if nums is None or m == 0:
            return 0
        # 初始化一个二维数组用于存储 背包在当前状态下最大的价值和
        f = [[0 for i in range(m+1)] for j in range(len(nums))]
        # 当背包容量是0，则背包能放入物品的最大体积也就是0，所以f[i][0] = 0
        # 当背包容量为j，则当j>=nums[0]时，才可以将第一个物品放入
        for j in range(m+1):
            if j >= nums[0]:
                f[0][j] = value[0]
        for i in range(1, len(nums)):
            for j in range(1, m+1):
                if j >= nums[i]:
                    # 此时考虑是否将nums[i]放入背包中，取是否放入的最大值
                    f[i][j] = max(f[i-1][j], f[i-1][j-nums[i]] + value[i])
                else:
                    f[i][j] = f[i-1][j]
        return f[len(nums)-1][m]

s = Solution()
nums = [5, 4, 7, 2, 6]
value = [12,3,10,3,6]
print(s.backPack(14, nums, value))

输出：
25

def helper(w, m, v):
    dp = [0 for i in range(w+1)]
    for i in range(len(m)):
        for j in range(w, m[i]-1, -1):
            dp[j] = max(dp[j], dp[j-m[i]] + v[i])
    print(dp)

def helper2(w, m, v):
    dp = [[0 for i in range(w+1)] for j in range(len(m))]
    for i in range(w+1):
        if i >= m[0]:
            dp[0][i] = v[0]
    for i in range(1, len(m)):
        for j in range(1, w+1):
            if j >= m[i]:
                dp[i][j] = max(dp[i-1][j], dp[i-1][j-m[i]] + v[i])
            else:
                dp[i][j] = dp[i - 1][j]
    print(dp)

if __name__ == '__main__':
    w = 10
    m = [1,2,3,4,5]
    v = [5,4,3,2,1]
    helper(w, m, v)
    helper2(w, m, v)
    
输出：
[0, 5, 5, 9, 9, 9, 12, 12, 12, 12, 14]
[[0, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5], 
 [0, 5, 5, 9, 9, 9, 9, 9, 9, 9, 9], 
 [0, 5, 5, 9, 9, 9, 12, 12, 12, 12, 12], 
 [0, 5, 5, 9, 9, 9, 12, 12, 12, 12, 14], 
 [0, 5, 5, 9, 9, 9, 12, 12, 12, 12, 14]]
```

### 考虑物品价值的背包问题（输出最优装入方案）
有容量w的船，有n个货物，m存储货物的重量，v存储货物的价值，求最大能装的价值，并输出装的货物，1代表装，0代表没装。

```python
def helper(w, m, v):
    dp = [[0 for i in range(w+1)] for j in range(len(m))]
    for i in range(w+1):
        if i >= m[0]:
            dp[0][i] = v[0]
    for i in range(1, len(m)):
        for j in range(1, w+1):
            if j >= m[i]:
                dp[i][j] = max(dp[i-1][j], dp[i-1][j-m[i]] + v[i])
            else:
                dp[i][j] = dp[i - 1][j]
    print(dp)
    j = w
    vis = [0 for i in range(len(m))]
    for i in range(len(m)-1, -1, -1):
        if i != 0:
            if dp[i][j] > dp[i-1][j]:
                vis[i] = 1
                j -= m[i]
            else:
                vis[i] = 0
        else:
            if dp[i][j] > 0:
                vis[i] = 1
                j -= m[i]
    print(vis)

if __name__ == '__main__':
    w = 10
    m = [1,2,3,4,5]
    v = [5,4,3,2,1]
    helper(w, m, v)
    
输出：
[[0, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5], 
 [0, 5, 5, 9, 9, 9, 9, 9, 9, 9, 9], 
 [0, 5, 5, 9, 9, 9, 12, 12, 12, 12, 12], 
 [0, 5, 5, 9, 9, 9, 12, 12, 12, 12, 14], 
 [0, 5, 5, 9, 9, 9, 12, 12, 12, 12, 14]]
[1, 1, 1, 1, 0]
```


## 完全背包
> 有`N`种物品和一个容量为`V`的背包，**每种物品都有无限件可用**。第`i`种物品的费用是`c[i]`，价值是`w[i]`。求解将哪些物品装入背包可使这些物品的费用总和不超过背包容量，且价值总和最大。 

这个问题非常类似于**01背包问题**，所不同的是每种物品有无限件。也就是从每种物品的角度考虑，与它**相关的策略已并非取或不取两种**，而是有**取0件、取1件、取2件……** 等很多种。如果仍然按照解01背包时的思路，令`f[i][v]`表示前`i`种物品恰放入一个容量为`v`的背包的最大价值总和。仍然可以按照每种物品不同的策略写出状态转移方程，像这样：

<img src="https://latex.codecogs.com/gif.latex?f[i][v]=max(f[i-1][v-k*c[i]]&plus;k*w[i]|0<=k*c[i]<=v)" title="f[i][v]=max(f[i-1][v-k*c[i]]+k*w[i]|0<=k*c[i]<=v)" />      

依照上面的递推式直接写程序是一个三重循环，时间复杂度很高，在这个算法中有多余的计算：在`f[i][v]`的计算中选择`k(k>=1)`个`i`物品的情况，与在`f[i][v-c[i]]`的计算中选择`k-1`个`i`物品的情况是相同的，所以`f[i][v]`的递推中`k>=1`部分的计算已经在`f[i][j-c[i]]`的计算中完成了。那么可以按照如下方式进行变形：   

<img src="https://latex.codecogs.com/gif.latex?\begin{align*}&space;f[i][v]&space;&=max(f[i-1][v-k*c[i]]&plus;k*w[i]|0<=k)\\&space;&&space;=max(f[i-1][v],max{f[i][v-k*c[i]]&plus;k*w[i]|1<=k})\\&space;&=max(f[i-1][v],max{f[i][(v-c[i])-k*c[i]]&plus;k*w[i]|0<=k}&plus;w[i])\\&space;&=max(f[i-1][v],f[i][v-c[i]]&plus;w[i])&space;\end{align*}" title="\begin{align*} f[i][v] &=max(f[i-1][v-k*c[i]]+k*w[i]|0<=k)\\ & =max(f[i-1][v],max{f[i][v-k*c[i]]+k*w[i]|1<=k})\\ &=max(f[i-1][v],max{f[i][(v-c[i])-k*c[i]]+k*w[i]|0<=k}+w[i])\\ &=max(f[i-1][v],f[i][v-c[i]]+w[i]) \end{align*}" />      

**基本解法**：
完全背包指每种物品有无数件，它和01背包的区别只有递推式的一个地方：
- 01 : <img src="https://latex.codecogs.com/gif.latex?\inline&space;f[i][j]&space;=&space;max(f[i-1][j],f[i-1][j-c[i]]&space;&plus;&space;v[i])" title="f[i][j] = max(f[i-1][j],f[i-1][j-c[i]] + v[i])" />
- 完全：<img src="https://latex.codecogs.com/gif.latex?\inline&space;f[i][j]&space;=&space;max(f[i-1][j],f[i][j-c[i]]&space;&plus;&space;v[i])" title="f[i][j] = max(f[i-1][j],f[i][j-c[i]] + v[i])" />

**解释**： 
考虑第`i`种物品，若不选，则`f[i][j] = f[i-1][j]`；若选择，意味着至少有一件`i`物品，考虑这一件物品，剩余`j-c[i]`空间用来放 `0...i`种 物品（`i`依然能放），这部分空间的最大价值为`f[i][j-c[i]]`。综合即可得到上述公式。

直观的，在递推时，01背包依赖的是**上一行的某两个格子（正上方和左侧）**，完全背包则依赖**上一行正上方的格子 + 同一行左侧的某个格子**。

```python
class Solution:
    # 使用二维数组
    def backPack(self, m, nums, value):
        if nums is None or m == 0:
            return 0
        # 初始化一个二维数组用于存储 背包在当前状态下最大的价值和
        f = [[0 for i in range(m+1)] for j in range(len(nums))]
        # 当背包容量是0，则背包能放入物品的最大体积也就是0，所以f[i][0] = 0
        # 当背包容量为j，则当j>=nums[0]时，才可以将第一个物品放入
        for j in range(m+1):
            if j >= nums[0]:
                f[0][j] = value[0]
        for i in range(0, len(nums)):
            for j in range(1, m+1):
                if j >= nums[i]:
                    f[i][j] = max(f[i-1][j], f[i][j-nums[i]]+value[i])
        return f[len(nums)-1][m]
        
    # 使用一维数组
    def backPack1(self, m, nums, value):
        if nums is None or m == 0:
            return 0
        # 初始化一个一维数组用于存储 背包在当前状态下最大的价值和
        f = [0 for i in range(m+1)]
        for i in range(0, len(nums)):
            for j in range(1, m+1):
                if j >= nums[i]:
                    f[j] = max(f[j], f[j-nums[i]]+value[i])
        return f[-1]
        
s = Solution()
nums = [5, 4, 7, 2, 6]
value=[12,3,10,3,6]
print(s.backPack(14, nums, value))
print(s.backPack1(14, nums, value))

输出：
30
30
```

## 多重背包
> 有`N`种物品和一个容量为`V`的背包，**第`i`种物品最多有`n[i]`件可用**。每件费用是`c[i]`，价值是`w[i]`。求解将哪些物品装入背包可使这些物品的费用总和不超过背包容量，且价值总和最大。 

# 凑硬币
[[编程题]拼凑钱币](https://www.nowcoder.com/questionTerminal/178b912722ac42a2865057a66d4e7de2?orderByHotValue=0&questionTypes=000100&page=1&onlyReference=false)
> 给你六种面额 1、5、10、20、50、100 元的纸币，假设每种币值的数量都足够多，编写程序求组成N元（N为0~10000的非负整数）的不同组合的个数。 

前`i`种硬币表示面额为`j`的方案数为`opt[j]= opt[j] + opt[j-nums[i]]`

```python
def coin(self, m, nums):
    if nums is None or m == 0:
        return 0
    # # 初始化一个二维数组用于存储 当前状态下的组合数
    # opt = [[0 for i in range(m+1)] for j in range(len(nums))]
    # for j in range(1,m+1):
    #     opt[0][j] = 1
    # for i in range(len(nums)):
    #     opt[i][0] = 1
    # for i in range(1,len(nums)):
    #     for j in range(1,m+1):
    #         if j >= nums[i]:
    #             opt[i][j] = opt[i-1][j] + opt[i][j-nums[i]]
    #         else:
    #             opt[i][j] = opt[i-1][j]
    # return opt[len(nums)-1][-1]
    # 初始化一维数组
    opt = [0 for i in range(m+1)]
    opt[0] = 1
    for i in range(0,len(nums)):
        # 这里也可以使用range(nums[i],m+1)，然后就不用做if判断了
        for j in range(1,m+1):
            if j >= nums[i]:
                opt[j] = opt[j] + opt[j-nums[i]]
            else:
                opt[j] = opt[j]
        print(opt)
    return opt[-1]
```

# 最长公共子序列和最长公共子串问题（LCS）
子序列是有序的，但不一定是连续，作用对象是序列。   
> 例如：序列 X = <B, C, D, B> 是序列 Y = <A, B, C, B, D, A, B> 的子序列，对应的下标序列为 <2, 3, 5, 7>。

子串是有序且连续的，左右对象是字符串。   
> 例如 a = abcd 是 c = aaabcdddd 的一个子串；但是 b = acdddd 就不是 c 的子串。

## 最长公共子序列
设输入序列是`X[0 .. m-1]`和`Y[0 .. n-1]`，长度分别为`m` 和 `n`。和设序列 `L(X [0 .. m-1]`，`Y[0 .. n-1])` 是这两个序列的 LCS 的长度，以下为 `L(X [0 .. M-1]`，`Y[0 .. N-1])` 的递归定义：   
1. 如果两个序列的最后一个元素匹配（即`X[M-1]` == `Y[N-1]`）   
    则：`L(X [0 .. M-1],Y [0 .. N-1])= 1 + L(X [0 .. M-2],Y [0 .. N-2])`
2. 如果两个序列的最后字符不匹配（即`X [M-1] != Y [N-1]`）   
    则：`L(X[0 .. M-1],Y[0 .. N-1]) = MAX(L(X[0 .. M-2],Y[0 .. N-1]),L(X[0 .. M-1],Y[0 .. N-2]))`

通过如下具体实例来更好地理解一下：
> 1. 考虑输入子序列 <AGGTAB> 和 <GXTXAYB>。最后一个字符匹配的字符串。这样的 LCS 的长度可以写成：    
L(<AGGTAB>, <GXTXAYB>) = 1 + L(<AGGTA>, <GXTXAY>)
> 2. 考虑输入字符串“ABCDGH”和“AEDFHR。最后字符不为字符串相匹配。这样的LCS的长度可以写成：   
L(<ABCDGH>, <AEDFHR>) = MAX ( L(<ABCDG>, <AEDFHR>), L(<ABCDGH>, <AEDFH>) )


```python
# 最长公共子序列
# 使用动态规划解决问题

def lcs(a,b):
    lena=len(a)
    lenb=len(b)
    c=[[0 for i in range(lenb+1)] for j in range(lena+1)]
    flag=[[0 for i in range(lenb+1)] for j in range(lena+1)]
    for i in range(lena):
        for j in range(lenb):
            # 这种写法也可以，这样就没有存储最长公共子序列的值
            # if a[i] == b[i]:
            #     c[i+1][j+1] = c[i][j] + 1
            # else:
            #     c[i+1][j+1] = max(c[i][j+1], c[i+1][j])
            if a[i]==b[j]:
                c[i+1][j+1]=c[i][j]+1
                flag[i+1][j+1]='ok'
            elif c[i+1][j]>c[i][j+1]:
                c[i+1][j+1]=c[i+1][j]
                flag[i+1][j+1]='left'
            else:
                c[i+1][j+1]=c[i][j+1]
                flag[i+1][j+1]='up'
    return c,flag

def printLcs(flag,a,i,j):
    if i==0 or j==0:
        return
    if flag[i][j]=='ok':
        printLcs(flag,a,i-1,j-1)
        print(a[i-1],end='')
    elif flag[i][j]=='left':
        printLcs(flag,a,i,j-1)
    else:
        printLcs(flag,a,i-1,j)

a='ABCBDAB'
b='BDCABA'
c,flag=lcs(a,b)
for i in c:
  print(i)
print('')
for j in flag:
  print(j)
print('')
printLcs(flag,a,len(a),len(b))
print('')

输出：
[0, 0, 0, 0, 0, 0, 0]
[0, 0, 0, 0, 1, 1, 1]
[0, 1, 1, 1, 1, 2, 2]
[0, 1, 1, 2, 2, 2, 2]
[0, 1, 1, 2, 2, 3, 3]
[0, 1, 2, 2, 2, 3, 3]
[0, 1, 2, 2, 3, 3, 4]
[0, 1, 2, 2, 3, 4, 4]

[0, 0, 0, 0, 0, 0, 0]
[0, 'up', 'up', 'up', 'ok', 'left', 'ok']
[0, 'ok', 'left', 'left', 'up', 'ok', 'left']
[0, 'up', 'up', 'ok', 'left', 'up', 'up']
[0, 'ok', 'up', 'up', 'up', 'ok', 'left']
[0, 'up', 'ok', 'up', 'up', 'up', 'up']
[0, 'up', 'up', 'up', 'ok', 'up', 'ok']
[0, 'ok', 'up', 'up', 'up', 'ok', 'up']

BCBA
```

## 最长公共子串
> 定义 2 个字符串 query 和 text, 如果 query 里最大连续字符子串在 text 中存在，则返回子串长度. 例如: query="acbac"，text="acaccbabb"， 则最大连续子串为 "cba", 则返回长度 3。

我们使用`c[i,j]` 表示 以 <img src="https://latex.codecogs.com/gif.latex?\inline&space;X_i" title="X_i" /> 和 <img src="https://latex.codecogs.com/gif.latex?\inline&space;Y_j" title="Y_j" /> 结尾的最长公共子串的长度，因为要求子串连续，所以对于 <img src="https://latex.codecogs.com/gif.latex?\inline&space;X_i" title="X_i" /> 与 <img src="https://latex.codecogs.com/gif.latex?\inline&space;Y_j" title="Y_j" /> 来讲，它们要么与之前的公共子串构成新的公共子串；要么就是不构成公共子串。故状态转移方程：   

```math
X[i-1] == Y[j-1],c[i,j] = c[i-1,j-1] + 1

X[i-1] != Y[j-1],c[i,j] = 0
```

```python
def find_lcsubstr(s1, s2):
    #生成0矩阵，为方便后续计算，比字符串长度多了一列
    m = [[0 for i in range(len(s2)+1)]  for j in range(len(s1)+1)]  
    mmax = 0   #最长匹配的长度
    p = 0  #最长匹配对应在s1中的最后一位
    for i in range(len(s1)):
        for j in range(len(s2)):
            if s1[i] == s2[j]:
                m[i+1][j+1] = m[i][j]+1
                if m[i+1][j+1] > mmax:
                    mmax = m[i+1][j+1]
                    p = i + 1
    return s1[p-mmax:p],mmax   #返回最长子串及其长度

print(find_lcsubstr('abcdfg','abdfg'))
```

# 最长递增子序列
[[编程题]最长递增子序列](https://www.nowcoder.com/questionTerminal/585d46a1447b4064b749f08c2ab9ce66)

参见[算法原型--最长递增子序列(Binary Search DP)](https://www.jianshu.com/p/25cc707d9c56)

**<img src="https://latex.codecogs.com/gif.latex?\inline&space;O(N^2)" title="O(N^2)" />解法**:

```python
def lis(arr):
    n = len(arr)
    # dp[i]表示在以arr[i]这个数结尾的情况下，arr[0....i]中的最大递增子序列
    dp = [1 for i in range(n)]
    for i in range(n):
        for j in range(i):
            if arr[i] > arr[j]:
                dp[i] = max(dp[i], dp[j] + 1)
    return dp

def generateLis(arr, dp):
    # 找出最大的值，便是最长递增子序列的长度
    n = max(dp)
    index = dp.index(n)
    res = [0 for i in range(n)]
    n -= 1
    res[n] = arr[index]
    # 从右向左
    for i in range(index, -1, -1):
        if arr[i] < arr[index] and dp[i] == dp[index] - 1:
            n -= 1
            res[n] = arr[i]
            index = i
    return res

if __name__ == '__main__':
    arr = [10, 22, 9, 33, 21, 50, 41, 60, 80]
    dp = lis(arr)
    print(dp)
    print("")
    res = generateLis(arr, dp)
    print(res)
    
输出：
[1, 2, 1, 3, 2, 4, 4, 5, 6]

[10, 22, 33, 41, 60, 80]
```

**<img src="https://latex.codecogs.com/gif.latex?\inline&space;O(n\log(n))" title="O(n\log(n))" />解法**
1. 使用一个数组h,首先令h[0]=arr[0]。记已经赋值了的h前部分为有序区，我们只考察有序区。
2. 往后遍历，对于arr[i],在h的有序区中寻找第一个大于arr[i]的位置。如果找到，就把那个位置的值更新为arr[i]，否则h的有序区长度增一，并且新增位置的值就为arr[i]。使用二分查找位置
3. 上述过程中，从位置0到，arr[i]的更新位置的元素个数就是以arr[i]结尾的最长递增子序列的长度。从二分查找出的位置就可以知道这个长度。使用一个全局变量来max存储更新最长递增子序列的长度。

**算法原理：**    
**h[i]表示遍历到当前时刻为止，长度为i+1的最长递增子序列的最小末尾。** 这样其实我们每次所做的工作就是要么增加了最长递增子序列的长度，要么就是长度不变，但是更新了每个长度对应的最小末尾，而这有利于之后扩展长度，因为你更小嘛，我后半的元素更容易比你大。这样其实最后的h的有效区长度即为所求，但是为了不再去遍历，中途使用一个max来记录当前位置时的最长递增子序列，更新max即可。做了n次,每次二分查找位置log(n),所以复杂度为n(logn)。

```python
class Solution(object):
    def lengthOfLIS(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        h = [0 for i in range(len(nums))]
        h[0] = nums[0]
        max = 0
        for i in range(len(nums)):
            if nums[i] > h[max]:
                h[max + 1] = nums[i]
                max += 1
            else:
                pos = self.findFirst(h, 0, max, nums[i])
                h[pos] = nums[i]
        return max + 1

    def findFirst(self, nums, left, right, target):
        if left == right:
            return left
        mid = (left + right) // 2
        if nums[mid] < target:
            return self.findFirst(nums, mid + 1, right, target)
        else:
            return self.findFirst(nums, left, mid, target)
```


# 最长回文子串
[LeetCode 题解 | 5. 最长回文子串](https://zhuanlan.zhihu.com/p/38251499)   
[[编程题]最长回文子串](https://www.nowcoder.com/questionTerminal/b4525d1d84934cf280439aeecc36f4af)   

```python
# 使用直接循环找出字符串中的回文子串，然后记录最长的回文子串，并输出
def solution1(self, s):
    mlen = len(s)
    if mlen == 1:
        return s
    maxlenght = 0  # 统计当前最长的回文子串的长度
    palindromic = []  # 统计当前最长的回文子串
    for i in range(mlen):
        for j in range(i+1, mlen):
            is_palindromic = True
            for k in range(i, ((i+j)//2)+1):
                if s[k] != s[j-k+i]:
                    is_palindromic = False
                    break
            if is_palindromic and j-i+1 > maxlenght:
                maxlenght = j - i + 1
                palindromic = s[i:j+1]
            # 也可以将所有的回文子串提取出来
            # if is_palindromic:
            #     palindromic.append(s[i:j + 1])

    return palindromic

# 使用中心扩展算法
# 通过枚举字符串子串的中心，并向两边同时扩散，依然是逐一判断子串的回文性。
def solution2(self, s):
    mlen = len(s)
    if mlen == 1:
        return s
    maxlength = 0
    start = 0  # 最长回文子串的起点
    for i in range(mlen):
        j = 0
        # 处理长度为偶数的回文子串
        while i-j >= 0 and i+j < mlen:
            if s[i-j] != s[i+j]:
                break
            if 2*j+1 > maxlength:
                maxlength = 2 * j + 1
                start = i - j
            j += 1
        j = 0
        # 处理长度是奇数的回文子串
        while i-j >= 0 and i+1+j < mlen:
            if s[i-j] != s[i+1+j]:
                break
            if 2*j+2 > maxlength:
                maxlength = 2 * j + 2
                start = i - j
            j += 1
    return s[start:start+maxlength]

# 动态规划方法
# dp[i,j] 表示第i到第j个字符是否是回文的
# dp[i,i]=true
# dp[i,j]=true, i到j的字符串是回文的
# dp[i,j]=false, i到j的字符串不是回文的
# dp[i,j]=(dp[i+1,j-1] and s[i] == s[j])
def solution3(self, s):
    mlen = len(s)
    if mlen == 1:
        return s
    dp = [[False for i in range(mlen)] for j in range(mlen)]
    dp[0][0] = True
    for i in range(1, mlen):
        dp[i][i] = True
        dp[i][i-1] = True  # 这个初始化容易忽略，当k=2时要用到
    maxlength = 0
    start = 0  # 最长回文子串的起点
    for k in range(2, mlen+1):  # 枚举子串的长度
        for i in range(mlen-k):  # 枚举子串的起始位置
            j = i + k - 1  # 枚举子串的结束位置
            if s[i] == s[j] and dp[i+1][j-1]:
                dp[i][j] = True
                start = i
                maxlength = k
    print(dp)
    return s[start:start+maxlength]
```

# 最长回文子序列
[算法导论学习之最长回文子序列](https://zhuanlan.zhihu.com/p/27365751)   
[(动态规划)最长回文子序列、回文子序列个数](http://www.cnblogs.com/AndyJee/p/4465696.html)    

```python
# 最长回文子序列
# 动态规划
# 对于任意字符串，如果头尾字符相同，那么字符串的最长子序列等于去掉首尾的字符串的最长子序列加上首尾；
# 如果首尾字符不同，则最长子序列等于去掉头的字符串的最长子序列和去掉尾的字符串的最长子序列的较大者。
# dp[i][j] 表示从i到j中最长回文子序列的长度值
# dp[i][i]=1
# dp[i][j]=dp[i+1][j-1] + 2  if（str[i]==str[j]）
# dp[i][j]=max(dp[i+1][j],dp[i][j-1])  if （str[i]!=str[j]）
# 计算dp[i][j]时需要计算dp[i+1][*]或dp[*][j-1]，因此i应该从大到小，即递减；j应该从小到大，即递增。
def lps(self, s):
    mlen = len(s)
    dp = [[0 for i in range(mlen)] for j in range(mlen)]
    for i in range(mlen):
        dp[i][i] = 1
    for i in range(mlen-1, -1, -1):
        for j in range(i+1, mlen):
            if s[i] == s[j]:
                dp[i][j] = dp[i + 1][j - 1] + 2
            else:
                dp[i][j] = max(dp[i + 1][j], dp[i][j - 1])
    return dp[0][-1]
```

# 377. 组合总和 Ⅳ
[组合总和 Ⅳ](https://leetcode-cn.com/problems/combination-sum-iv/description/)

> 给定一个由正整数组成且不存在重复数字的数组，找出和为给定目标正整数的组合的个数。    
nums = [1, 2, 3]     
target = 4     
所有可能的组合为：    
(1, 1, 1, 1)    
(1, 1, 2)    
(1, 2, 1)   
(1, 3)    
(2, 1, 1)    
(2, 2)   
(3, 1)    
请注意，顺序不同的序列被视作不同的组合。   
因此输出为 7。  

使用动态规划和递归均可，创建一个dp数组，dp[i]表示和为i的正整数组合的个数，dp[0]=1,则从i=1到target遍历，对每一个i遍历数组中每个num，若i>=num,则dp[i]+=dp[i-num],表示dp[3]=dp[2]+1 或 dp[1]+2 或 dp[0]+3,将所有情况累加就是dp[3]的结果,对原数组排序可对算法进行优化，当i< num后面则不用判断直接break。

```python
class Solution:
    def combinationSum4(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        # 动态规划
        # 创建一个dp数组，dp[i]表示和为i的正整数组合的个数，dp[0]=1,
        # 则从i=1到target遍历，对每一个i遍历数组中每个num
        nums.sort()
        dp = [0 for i in range(target+1)]
        dp[0] = 1
        for i in range(target+1):
            for val in nums:
                if i < val:
                    break
                dp[i] = dp[i] + dp[i-val]
        return dp[target]
```

# 132. 分割回文串 II
[分割回文串 II](https://leetcode-cn.com/problems/palindrome-partitioning-ii/description/)
> 给定一个字符串 s，将 s 分割成一些子串，使每个子串都是回文串。
返回符合要求的最少分割次数。  

解题思路参见[Palindrome Partitioning II 分割回文串 II](http://xin.fmachine.cn/2016/11/08/Palindrome-Partitioning-II/)   
python会超时，可以选用Java

```python
class Solution:
    def partition(self, s):
        """
        :type s: str
        :rtype: List[List[str]]
        """
        # 动态规划
        # dp[i] 表示 子串(0,i)之间最小回文切割
        # 1. 当(0,i)本来就是回文串，则dp[i]=0,否则，dp[i] = i（表示至多分割i次）;
        # 2. 对于任意大于1的i，1<=j<=i, 如果(j,i)是回文串，则dp[i] = min(dp[i], dp[j-1]+1)
        if len(s) < 2:
            return 0
        dp = [i for i in range(len(s))]
        for i in range(1, len(s)):
            if self.ispalindromic(s, 0, i):
                dp[i] = 0
            for j in range(i, 0, -1):
                if self.ispalindromic(s, j, i):
                    dp[i] = min(dp[i], dp[j-1] + 1)
        return dp[-1]

    def ispalindromic(self, s, start, end):
        # print(start, end)
        while start < end:
            if s[start] != s[end]:
                return False
            start += 1
            end -= 1
        return True
```

```java
class Solution {
     public int minCut(String s) {
        if(s == null||s.length() == 0)
            return 0;
        int[] dp=new int[s.length()];
        //dp[i]存放(0,i)即以i的字符结束的子串的最小切割数，则所求为dp[s.length()-1];
        dp[0]=0;//一个字符，不需要切割
        for(int i=1;i<s.length();i++) {
            //dp[i]赋初值
            dp[i]=is_palindrome(s.substring(0,i+1))?0:i+1;
            //  1=<j<=i的子串回文判定
            for(int j=i;j>=1;j--) {
                if(is_palindrome(s.substring(j,i+1))) {
                  dp[i]=Math.min(dp[i],dp[j-1]+1);
                }
            }
        }
        return dp[s.length()-1];
    }
    //判断回文串例程
    public boolean is_palindrome(String s) {
        int begin=0;
        int end=s.length()-1;
        while(begin<end) {
            if(s.charAt(begin)!=s.charAt(end))
                return false;
            begin++;
            end--;            
        }
        return true;
    }

}
```

优化版Java：

```java
class Solution {
    public int minCut(String s) {
        if(s.length()<2)
            return 0;
        int len = s.length();
        // 表示从i到字符串为可以被最小分割的次数
        int dp[] = new int[len+1];
        // 存储字符串从i到j是否是回文串
        boolean isPalindrome[][] = new boolean[len+1][len+1];
        for(int i = 0;i<len;i++){
            dp[i] = len-i-1;  // 先初始化分割次数
            isPalindrome[i][i] = true;
        }
        dp[len] = -1;
        for(int i = len-1;i>=0;i--){
            for(int j = i;j<len;j++){
                if((s.charAt(i)==s.charAt(j))&&(i+1>=j||isPalindrome[i+1][j-1])){
                    dp[i] = Math.min(dp[i],dp[j+1]+1);
                    isPalindrome[i][j] = true;
                }
            }
        }
        return dp[0];
    }
}
```



# 参考文献
[动态规划：从新手到专家](http://www.hawstein.com/posts/dp-novice-to-advanced.html)   
[动态规划之背包问题（一）](http://www.hawstein.com/posts/dp-knapsack.html)   
[五大常用算法之二：动态规划算法](https://www.cnblogs.com/steven_oyj/archive/2010/05/22/1741374.html)   
[【算法】动态规划问题集锦与讲解](https://segmentfault.com/a/1190000004498566#articleHeader3)   
[LCSs——最长公共子序列和最长公共子串](http://www.cnblogs.com/maybe2030/p/5469877.html)   
[背包问题](http://novoland.github.io/%E7%AE%97%E6%B3%95/2014/07/26/%E8%83%8C%E5%8C%85%E9%97%AE%E9%A2%98.html#csaww)    
[背包之01背包、完全背包、多重背包详解](http://www.wutianqi.com/?p=539)   
[完全背包问题（动态规划（DP））](https://blog.csdn.net/Zhao_Xinhao/article/details/77153300)   
[01背包、完全背包、多重背包、二维背包之Python实现](https://zhuanlan.zhihu.com/p/35643721)    
[算法设计 —— LCS 最长公共子序列&&最长公共子串 &&LIS 最长递增子序列](https://segmentfault.com/a/1190000002641054#articleHeader5)   
[算法题解：经典的动态规划问题——最长递增子序列（一）](https://segmentfault.com/a/1190000012748540)   
[每天一道编程题——最长递增子序列](https://wax8280.github.io/2016/10/18/859/)   
[最长递增子序列LIS的O(nlogn)的求法](https://blog.csdn.net/u012505432/article/details/52228945)   
[LeetCode 5. Longest Palindromic Substring（最长回文子串）](https://www.jianshu.com/p/7c3f074b380b)    
[Leetcode-5-Longest Palindromic Substring](http://zyy1217.com/2017/04/09/leetcode5/)   
[LeetCode 5. Longest Palindromic Substring 最长回文子串 Python 四种解法(Manacher 动态规划)](https://blog.csdn.net/asd136912/article/details/78987624)    
[LeetCode:Longest Palindromic Substring 最长回文子串](http://www.cnblogs.com/TenosDoIt/p/3675788.html)   
[算法导论学习之最长回文子序列](https://zhuanlan.zhihu.com/p/27365751)    