[toc]

# 和深度相关的问题
## 104. 二叉树的最大深度
[二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/description/)    
> 给定一个二叉树，找出其最大深度。     
二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。

**方法1：递归实现**
```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def maxDepth(self, root):
        """
        :type root: TreeNode
        :rtype: int
        """
        if not root:
            return 0
        left = self.maxDepth(root.left)
        right = self.maxDepth(root.right)
        return max(left, right) + 1
```

**方法2：非递归实现**     
使用层次遍历，将每一层的节点信息存在一个list中。

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def maxDepth(self, root):
        """
        :type root: TreeNode
        :rtype: int
        """
        if not root:
            return 0
        # 存储层次遍历的结果
        res = []
        # 用队列来存储访问过的节点，辅助完成层次遍历
        queue = []
        # 表示当前访问的节点
        node = root
        # 统计深度
        count = 0
        queue.append(node)
        while queue:
            # 用于存储同层的节点
            tmp = []
            for i in range(len(queue)):
                # 将同层节点依次出队列
                cur = queue.pop(0)
                if cur.left:
                    queue.append(cur.left)
                if cur.right:
                    queue.append(cur.right)
                tmp.append(cur.val)
            if len(tmp) > 0:
                count += 1
            res.append(tmp)
        return count
```

## 111. 二叉树的最小深度
[二叉树的最小深度](https://leetcode-cn.com/problems/minimum-depth-of-binary-tree/description/)   
> 给定一个二叉树，找出其最小深度。   
最小深度是从根节点到最近叶子节点的最短路径上的节点数量。    
说明: 叶子节点是指没有子节点的节点。   

将二叉树分为这么几种情况：
- 传入的根节点为空，返回0；
- 传入根节点不为空，左子树为空，右子树为空，返回最小深度1；
- 传入根节点不为空，左子树为空，右子树不为空，返回右子树的最小深度+1；
- 传入根节点不为空，左子树不为空，右子树为空，返回左子树的最小深度+1；
- 传入根节点不为空，左右子树都不为空，则返回左右子树中最小深度的较小值+1.

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def minDepth(self, root):
        """
        :type root: TreeNode
        :rtype: int
        """
        if not root:
            return 0
        if not root.left and not root.right:
            return 1
        elif not root.left and root.right:
            return self.minDepth(root.right) + 1
        elif root.left and not root.right:
            return self.minDepth(root.left) + 1
        elif root.left and root.right:
            left = self.minDepth(root.left)
            right = self.minDepth(root.right)
            return min(left, right) + 1
```


## 110. 平衡二叉树
[平衡二叉树](https://leetcode-cn.com/problems/balanced-binary-tree/description/)
> 给定一个二叉树，判断它是否是高度平衡的二叉树。     
本题中，一棵高度平衡二叉树定义为：**一个二叉树每个节点 的左右两个子树的高度差的绝对值不超过1**。

直接定义一个求高度的函数。若子树不平衡，则返回-1。

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    # 使用求深度函数，求每个节点的左右子树的高度差，如果高度差的绝对值大于1
    # 则返回-1，代表不平衡
    def isBalanced(self, root):
        """
        :type root: TreeNode
        :rtype: bool
        """
        isB = self.height(root)
        return isB != -1
        
    def height(self, root):
        if not root:
            return 0
        left = self.height(root.left)
        right = self.height(root.right)
        if left == -1 or right == -1 or abs(left - right) > 1:
            return -1
        return max(left, right) + 1
```

## 559. N叉树的最大深度
[N叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-n-ary-tree/description/)
> 给定一个N叉树，找到其最大深度。    
最大深度是指从根节点到最远叶子节点的最长路径上的节点总数。

通过递归，找到每个子节点的最大深度。然后给子节点的最大深度+1就可以得到该节点的最大深度。

```python
"""
# Definition for a Node.
class Node(object):
    def __init__(self, val, children):
        self.val = val
        self.children = children
"""
class Solution(object):
    def maxDepth(self, root):
        """
        :type root: Node
        :rtype: int
        """
        if not root:
            return 0
        depth = 0
        for node in root.children:
            depth = max(depth, self.maxDepth(node))
        return depth + 1
```

# 二叉树路径和相关问题
## 112. 路径总和
[路径总和](https://leetcode-cn.com/problems/path-sum/description/)  
> 给定一个二叉树和一个目标和，判断该树中是否存在根节点到叶子节点的路径，这条路径上所有节点值相加等于目标和。    
说明: 叶子节点是指没有子节点的节点。

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def hasPathSum(self, root, sum):
        """
        :type root: TreeNode
        :type sum: int
        :rtype: bool
        """
        if not root:
            return False
        # 是叶子结点且路径综总和满足
        if not root.left and not root.right and root.val == sum:
            return True
        if self.hasPathSum(root.left, sum - root.val):
            return True
        if self.hasPathSum(root.right, sum - root.val):
            return True
        return False
```

## 113. 路径总和 II
[路径总和 II](https://leetcode-cn.com/problems/path-sum-ii/description/)
> 给定一个二叉树和一个目标和，找到所有从根节点到叶子节点路径总和等于给定目标和的路径。

tmp用来保存每一条路径的值，当节点到达叶子节点且节点值和sum值相同的时候，就把tmp的值压入res中。每一次的dfs的遍历中止与叶子结点，访问到叶子结点时，需要把叶子结点弹出。

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def pathSum(self, root, sum):
        """
        :type root: TreeNode
        :type sum: int
        :rtype: List[List[int]]
        """
        res = []
        tmp = []
        self.helper(root, tmp, res, sum)
        return res
    
    def helper(self, root, tmp, res, sum):
        if not root:
            return
        tmp.append(root.val)
        if not root.left and not root.right and root.val == sum:
            res.append(tmp[:])
        self.helper(root.left, tmp, res, sum-root.val)
        self.helper(root.right, tmp, res, sum-root.val)
        tmp.pop()
```

## 437. 路径总和 III
[路径总和 III](https://leetcode-cn.com/problems/path-sum-iii/description/)
> 给定一个二叉树，它的每个结点都存放着一个整数值。      
找出路径和等于给定数值的路径总数。     
路径不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。

使用深度优先遍历，找从每个节点出发符合要求的路径数。

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def pathSum(self, root, sum):
        """
        :type root: TreeNode
        :type sum: int
        :rtype: int
        """
        if not root:
            return 0
        return self.helper(root, sum) + self.pathSum(root.left, sum) + self.pathSum(root.right, sum)
        
    def helper(self, root, sum):
        res = 0
        if not root:
            return res
        if root.val == sum:
            res += 1
        res += self.helper(root.left, sum - root.val)
        res += self.helper(root.right, sum - root.val)
        return res
```

## 129. 求根到叶子节点数字之和
[求根到叶子节点数字之和](https://leetcode-cn.com/problems/sum-root-to-leaf-numbers/description/)
> 给定一个二叉树，它的每个结点都存放一个0-9的数字，每条从根到叶子节点的路径都代表一个数字。    
例如，从根到叶子节点路径 1->2->3 代表数字 123。   
计算从根到叶子节点生成的所有数字之和。

类似于找出树每条路径。这里需要把路径中的值组成一个数，然后求和。同样使用dfs。

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def sumNumbers(self, root):
        """
        :type root: TreeNode
        :rtype: int
        """
        res = []
        self.helper(root, '', res)
        res = list(map(int, res))
        return sum(res)
        
    def helper(self, root, tmp, res):
        if not root:
            return 
        if not root.left and not root.right:
            res.append(tmp + str(root.val))
        self.helper(root.left, tmp + str(root.val), res)
        self.helper(root.right, tmp + str(root.val), res)
```

## 687. 最长同值路径
[最长同值路径](https://leetcode-cn.com/problems/longest-univalue-path/description/)
> 给定一个二叉树，找到最长的路径，这个路径中的每个节点具有相同值。 这条路径可以经过也可以不经过根节点。

解答参见[687. 最长同值路径（和543相似，返回值不同，因为意义不同）](https://blog.csdn.net/xuchonghao/article/details/80692887)
1. 首先对其左右孩子调用递归函数，得到其左右孩子的最大同值路径，然后根据其与左右孩子的关系来判断结果。 
2. 如果root的值与其左孩子的值相同，那么从左孩子得到的路径值需要加一；如果不相同，那么置零。右孩子同理。 
3. 然后最大值max就是当前的最大值或者左右孩子路径的和。 
4. 返回值是左右路径中的最大值，因为它还需要和父节点继续构建路径。

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def __init__(self):
        self.maxnum = 0  # 用于记录每个节点的最长路径
        
    def longestUnivaluePath(self, root):
        """
        :type root: TreeNode
        :rtype: int
        """
        self.helper(root)
        return self.maxnum
    
    def helper(self, root):
        if not root:
            return 0
        left = self.helper(root.left)
        right = self.helper(root.right)
        if root.left and root.left.val == root.val:
            left += 1
        else:
            left = 0
        if root.right and root.right.val == root.val:
            right += 1
        else:
            right = 0
        self.maxnum = max(self.maxnum, left + right)
        return max(left, right)
```

## 124. 二叉树中的最大路径和
[二叉树中的最大路径和](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/description/)
> 给定一个非空二叉树，返回其最大路径和。    
本题中，路径被定义为一条从树中任意节点出发，达到任意节点的序列。该路径至少包含一个节点，且不一定经过根节点。

解答参见[LeetCode 124. 二叉树中的最大路径和](https://blog.csdn.net/Koala_Tree/article/details/80109258)
- 以递归的思想，深度优先，从下到上寻找最大的路径和；
- 对于每个节点可以与其左右节点相结合，但是当每个根结点作为左（右）子节点返回时，只能选择该根根结点和其左右子节点中的最大的一个。（这样才能称为路径）

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

import sys
class Solution(object):
    def __init__(self):
        self.maxsum = -sys.maxsize-1  
    def maxPathSum(self, root):
        """
        :type root: TreeNode
        :rtype: int
        """
        self.helper(root)
        return self.maxsum
    
    def helper(self, root):
        if not root:
            return 0
        left = max(0, self.helper(root.left))
        right = max(0, self.helper(root.right))
        self.maxsum = max(self.maxsum, left + right + root.val)
        return root.val + max(left, right)
```



# 二叉树遍历相关问题
## 107. 二叉树的层次遍历 II
[二叉树的层次遍历 II](https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii/description/)
> 给定一个二叉树，返回其节点值自底向上的层次遍历。 （即按从叶子节点所在层到根节点所在的层，逐层从左向右遍历）

可以使用层次遍历，然后逆序输出。    
也可以使用层次遍历的方式每次向list[0]的位置插入每层的数值。

**方法一：递归方式**
```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def levelOrderBottom(self, root):
        """
        :type root: TreeNode
        :rtype: List[List[int]]
        """
        level, res = 1, []
        self.helper(level, res, root)
        return res
    
    def helper(self, level, res, root):
        if root:
            # 每次向list的0位置上插入一个空的list,用于保存下一层的数值
            if level > len(res):
                res.insert(0, [])
            res[-level].append(root.val)
            self.helper(level+1, res, root.left)
            self.helper(level+1, res, root.right)
```

**方法二：非递归，使用队列**

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def levelOrderBottom(self, root):
        """
        :type root: TreeNode
        :rtype: List[List[int]]
        """
        if not root:
            return []
        queue = []
        res = []
        cur = root
        queue.append(cur)
        while queue:
            tmp = []
            for i in range(len(queue)):
                node = queue.pop(0)
                if node.left:
                    queue.append(node.left)
                if node.right:
                    queue.append(node.right)
                tmp.append(node.val)
            res.insert(0, tmp)
        return res
```

## 103. 二叉树的锯齿形层次遍历
[二叉树的锯齿形层次遍历](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/description/)
> 给定一个二叉树，返回其节点值的锯齿形层次遍历。（即先从左往右，再从右往左进行下一层遍历，以此类推，层与层之间交替进行）。

还是层次遍历的思路，然后设置一个标志位用来标记某一层是否需要反转。

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def zigzagLevelOrder(self, root):
        """
        :type root: TreeNode
        :rtype: List[List[int]]
        """
        if not root:
            return []
        queue = []
        res = []
        # 标志是否需要反转
        flag = -1
        cur = root
        queue.append(cur)
        while queue:
            tmp = []
            for i in range(len(queue)):
                node = queue.pop(0)
                if node.left:
                    queue.append(node.left)
                if node.right:
                    queue.append(node.right)
                tmp.append(node.val)
            if flag > 0:
                res.append(tmp[::-1])
                flag = -1
            else:
                res.append(tmp)
                flag = 1
        return res
```

# 114. 二叉树展开为链表
[二叉树展开为链表](https://leetcode-cn.com/problems/flatten-binary-tree-to-linked-list/description/)    
> 给定一个二叉树，原地将它展开为链表。

参见[LeetCode（114）： 二叉树展开为链表](https://www.cnblogs.com/ariel-dreamland/p/9162548.html)

这道题要求把二叉树展开成链表，根据展开后形成的链表的顺序分析出是使用**先序遍历**，那么只要是数的遍历就有递归和非递归的两种方法来求解，这里我们也用两种方法来求解。

首先来看**递归版本**的，思路是先利用DFS的思路找到最左子节点，然后回到其父节点，把其父节点和右子节点断开，将原左子结点连上父节点的右子节点上，然后再把原右子节点连到新右子节点的右子节点上，然后再回到上一父节点做相同操作。

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def flatten(self, root):
        """
        :type root: TreeNode
        :rtype: void Do not return anything, modify root in-place instead.
        """
        if not root:
            return
        # 先向左找
        if root.left:
            self.flatten(root.left)
        # 先向右找
        if root.right:
            self.flatten(root.right)
        # 将左子节点移到右子节点
        tmp = root.right
        root.right = root.left
        root.left = None
        # 向前移动指针
        while root.right:
            root = root.right
        # 将原来的右子节点接在当前的右子节点后面
        root.right = tmp
```

下面我们再来看**非递归版本**的实现，这个方法是：
1. 从根节点开始出发，先检测其左子结点是否存在
2. 如存在则将根节点和其右子节点断开
3. 将左子结点及其后面所有结构一起连到原右子节点的位置，把原右子节点连到原左子结点最后面的右子节点之后。

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def flatten(self, root):
        """
        :type root: TreeNode
        :rtype: void Do not return anything, modify root in-place instead.
        """
        cur = root
        while cur:
            if cur.left:
                # 将左子节点加到右子节点和根节点之间
                p = cur.left
                while p.right:
                    p = p.right
                p.right = cur.right
                cur.right = cur.left
                cur.left = None
            cur = cur.right
```

# 257. 二叉树的所有路径
[二叉树的所有路径](https://leetcode-cn.com/problems/binary-tree-paths/description/)
> 给定一个二叉树，返回所有从根节点到叶子节点的路径。

深度优先遍历。

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def binaryTreePaths(self, root):
        """
        :type root: TreeNode
        :rtype: List[str]
        """
        res = []
        self.helper(root, '', res)
        return res
        
    def helper(self, root, tmp, res):
        if not root:
            return 
        
        if not root.left and not root.right:
            res.append(tmp + str(root.val))
        self.helper(root.left, tmp + str(root.val) + '->', res)
        self.helper(root.right, tmp + str(root.val) + '->', res)
```

# 543. 二叉树的直径
[543. 二叉树的直径](https://leetcode-cn.com/problems/diameter-of-binary-tree/description/)     
> 给定一棵二叉树，你需要计算它的直径长度。一棵二叉树的直径长度是任意两个结点路径长度中的最大值。这条路径可能穿过根结点。

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def diameterOfBinaryTree(self, root):
        """
        :type root: TreeNode
        :rtype: int
        """
        if not root:
            return 0
        res = self.maxDepth(root.left) + self.maxDepth(root.right)
        return max(res, self.diameterOfBinaryTree(root.left), self.diameterOfBinaryTree(root.right))

    def maxDepth(self, root):
        if not root:
            return 0
        left = self.maxDepth(root.left)
        right = self.maxDepth(root.right)
        return max(left, right) + 1
```



# 求二叉树中节点的最大距离
[求二叉树中节点的最大距离](https://segmentfault.com/a/1190000002616061)   
[《编程之美: 求二叉树中节点的最大距离》的另一个解法](https://www.cnblogs.com/miloyip/archive/2010/02/25/binary_tree_distance.html)

# 450. 删除二叉搜索树中的节点
[450. 删除二叉搜索树中的节点](https://leetcode-cn.com/problems/delete-node-in-a-bst/)
> 给定一个二叉搜索树的根节点 root 和一个值 key，删除二叉搜索树中的 key 对应的节点，并保证二叉搜索树的性质不变。返回二叉搜索树（有可能被更新）的根节点的引用。   
一般来说，删除节点可分为两个步骤：   
1.首先找到需要删除的节点；   
2.如果找到了，删除它。

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def deleteNode(self, root, key):
        """
        :type root: TreeNode
        :type key: int
        :rtype: TreeNode
        """
        if not root:
            return None
        if root.val < key:
            root.right = self.deleteNode(root.right, key)
            return root
        elif root.val > key:
            root.left = self.deleteNode(root.left, key)
            return root
        else:
            if not root.left:
                return root.right
            if not root.right:
                return root.left
            tmp = self.getMin(root.right)
            tmp.right = self.removeMin(root.right)
            tmp.left = root.left
            return tmp
            
                
    def removeMin(self, root):
        if not root.left:
            right = root.right
            return right

        root.left = self.removeMin(root.left)
        return root
    
    def getMin(self, root):
        if root.left:
            self.getMin(root.left)
        else:
            return root
```


