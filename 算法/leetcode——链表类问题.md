* [237. 删除链表中的节点](#237-删除链表中的节点)
* [203. 删除链表中的节点](#203-删除链表中的节点)
* [反转链表](#反转链表)
    * [206. 反转链表](#206-反转链表)
    * [92. 反转链表 II](#92-反转链表-ii)
* [234. 回文链表](#234-回文链表)
* [链表排序](#链表排序)
    * [147. 对链表进行插入排序](#147-对链表进行插入排序)
    * [148. 排序链表](#148-排序链表)
        * [快速排序](#快速排序)
        * [归并排序](#归并排序)
* [相交链表问题](#相交链表问题)
    * [160. 相交链表](#160-相交链表)
* [环形链表问题](#环形链表问题)
    * [141. 环形链表](#141-环形链表)
    * [142. 环形链表 II](#142-环形链表-ii)
* [删除链表中的重复元素](#删除链表中的重复元素)
    * [83. 删除排序链表中的重复元素](#83-删除排序链表中的重复元素)
    * [82. 删除排序链表中的重复元素 II](#82-删除排序链表中的重复元素-ii)
* [61. 旋转链表](#61-旋转链表)
* [参考文献](#参考文献)

# 237. 删除链表中的节点
[删除链表中的节点](https://leetcode-cn.com/problems/delete-node-in-a-linked-list/description/)    

> 请编写一个函数，使其可以删除某个链表中给定的（非末尾）节点，只允许访问该节点

通过访问该节点，让下个节点的内容覆盖该节点就可以了。


```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def deleteNode(self, node):
        """
        :type node: ListNode
        :rtype: void Do not return anything, modify node in-place instead.
        """
        tmp = node.next
        node.val = tmp.val
        node.next = tmp.next
```

# 203. 删除链表中的节点
[删除链表中的节点](https://leetcode-cn.com/problems/remove-linked-list-elements/description/)     
> 删除链表中等于给定值 val 的所有节点。

1. 考虑链表头部出现相等的情况
2. 考虑head为None的情况
3. 最后用next节点替换当前相等的节点

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def removeElements(self, head, val):
        """
        :type head: ListNode
        :type val: int
        :rtype: ListNode
        """
        while head and head.val == val:
            tmp = head
            head = tmp.next
        if not head:
            return head
        cur = head
        while cur.next:
            tmp = cur.next
            if tmp.val == val:
                cur.next = tmp.next
            else:
                cur = cur.next
        return head
```

也可以通过建立虚拟头节点的方式解决这个问题。

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def removeElements(self, head, val):
        """
        :type head: ListNode
        :type val: int
        :rtype: ListNode
        """
        h = ListNode(-1)
        h.next = head
        cur = h
        while cur.next != None:
            tmp = cur.next
            if tmp.val == val:
                cur.next = tmp.next
            else:
                cur = cur.next
        return h.next
```

# 反转链表
## 206. 反转链表
[反转链表](https://leetcode-cn.com/problems/reverse-linked-list/description/) 
> 反转一个单链表。  

解法参见[算法题：链表反转](https://foofish.net/linklist-reverse.html)    

非递归版本：
```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def reverseList(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        if head is None or head.next is None:
            return head
        preNode = None
        while head != None:
            nextNode = head.next  # 缓存当前节点的向后指针，待下次迭代用
            head.next = preNode  # 这一步是反转的关键，相当于把当前的向前指针作为当前节点的向后指针
            preNode = head  # 作为下次迭代时的（当前节点的）向前指针
            head = nextNode   # 作为下次迭代时的（当前）节点
        # 返回头指针，头指针就是迭代到最后一次时的head变量（赋值给了pre）,
        # 因为最后一次head指向了None，所以不能返回head
        return preNode
            
```
递归版本：

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def reverseList(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        if head is None or head.next is None:
            return head
        newhead = self.reverseList(head.next)  # 先反转后面的链表，从最后面的两个节点开始反转，依次向前
        head.next.next = head  # 将后一个节点指向当前节点
        head.next = None  # 将原链表中当前节点和后一个节点的连接断开
        return newhead
```

## 92. 反转链表 II
[反转链表 II](https://leetcode-cn.com/problems/reverse-linked-list-ii/description/)    
> 反转从位置 m 到 n 的链表。请使用一趟扫描完成反转。

参见[Leetcode 92:反转链表 II（最详细解决方案！！！）](https://blog.csdn.net/qq_17550379/article/details/80649221)

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def reverseBetween(self, head, m, n):
        """
        :type head: ListNode
        :type m: int
        :type n: int
        :rtype: ListNode
        """
        if not head or not head.next or m == n:
            return head
        newhead = ListNode(-1)
        newhead.next = head
        pre = newhead
        cur = head
        i = 1
        # 找到起始节点
        while i < m:
            pre = cur
            cur = cur.next
            i += 1
        t1 = pre  # 表示需要反转的第一个节点的前一个节点
        t2 = cur  # 表示需要反转的第一个节点，也是反转后的最后一个节点
        while i <= n:
            tmp = cur.next
            cur.next = pre
            pre = cur
            cur = tmp
            i += 1
        t1.next = pre
        t2.next = cur
        return newhead.next
```

# 234. 回文链表
[回文链表](https://leetcode-cn.com/problems/palindrome-linked-list/description/)
> 请判断一个链表是否为回文链表。O(n) 时间复杂度和 O(1) 空间复杂度

1. 快慢指针： slow 和 fast，每次slow指针前进一步，fast指针前进两步，当遇到指针为空时说明遍历链表完成，此时也就可以找到链表的中心位置。注意，由于**链表的长度可能是奇数也可能是偶数**，因此应该做一个判断。
    - 奇数：slow恰好在中间节点，fast恰好在最后一个节点。slow.next就是下一个链表的头节点。你会发现这两个链表是不一样的长的，但是在比较的时候，只要有一个链表是None的时候就结束比较的，也就是说只与第二个链表有关系的。   
    - 偶数：slow在N/2的**向下取整处**的节点，也就是中间两个节点的前一个，而fast在倒数第二个节点，下面就一样了。
2. 找到链表的中心后，把链表的后半部分进行就地逆转，就地逆转是采用头插法即可。
3. 后半部分逆转完成后，将链表的前半部分和后半部分一次比较，即可判断是否是回文。

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def isPalindrome(self, head):
        """
        :type head: ListNode
        :rtype: bool
        """
        if head is None or head.next is None:
            return True
        # 找到中心节点slow
        slow = head
        fast = head
        while fast.next != None and fast.next.next != None:
            slow = slow.next
            fast = fast.next.next
        # 将链表断开成两个链表
        newhead = slow.next
        # 将后一个链表反转
        head1 = self.reverseLink(newhead)
        # 对比两个链表的值是否相等
        # 当遇到一个出现 None 则跳出循环
        while head and head1:
            if head.val != head1.val:
                return False
            head = head.next
            head1 = head1.next
        return True


    def reverseLink(self, head):
        if head == None or head.next == None:
            return head
        prenode = None
        while head:
            nextnode = head.next
            head.next = prenode
            prenode = head
            head = nextnode
        return prenode
```

# 链表排序
## 147. 对链表进行插入排序
[对链表进行插入排序](https://leetcode-cn.com/problems/insertion-sort-list/description/)     

详解参见[leetcode题解-链表排序算法 147. Insertion Sort List && 148 . Sort List](https://blog.csdn.net/liuchonge/article/details/74394995)   

版本1：每次遍历从排序链表头部开始寻找
```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def insertionSortList(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        if not head:
            return None
        # 定义一个新的链表用于存储排序好的链表
        newhead = ListNode(-1)
        # cur用于遍历要排序的链表，pre指向排序号的链表，nextnode暂存剩下的链表
        cur = head
        pre = newhead
        while cur:
            nextnode = cur.next
            # 一般都选择在当前元素后面插入，因为这样比较方便，也不会出现边界异常的现象
            while pre.next and pre.next.val < cur.val:
                pre = pre.next
            # 将cur插入到相应位置
            cur.next = pre.next
            pre.next = cur
            # 恢复pre和cur的值
            pre = newhead
            cur = nextnode
        return newhead.next
```
版本2：优化版，在上一次插入后不把pre定向到排序链表的头部，一个元素到来时直接从该位置开始遍历，如果pre的值比较大，那么久返回newHead重新遍历，如果pre的值小于cur，那就直接向后遍历，这样就省去了从头到pre的遍历时间。

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None
class Solution(object):
    def insertionSortList(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        if not head:
            return None
        # 定义一个新的链表用于存储排序好的链表
        newhead = ListNode(-1)
        # cur用于遍历要排序的链表，pre指向排序号的链表，nextnode暂存剩下的链表
        cur = head
        pre = newhead
        while cur:
            nextnode = cur.next
            # 只有当有序链表中当前节点的值大于待排序链表的当前节点时，才会将有序链表的指针移到表头
            if pre != newhead and pre.val > cur.val:
                pre = newhead
            # 一般都选择在当前元素后面插入，因为这样比较方便，也不会出现边界异常的现象
            while pre.next and pre.next.val < cur.val:
                pre = pre.next
            cur.next = pre.next
            pre.next = cur
            cur = nextnode
        return newhead.next
```

## 148. 排序链表
[排序链表](https://leetcode-cn.com/problems/sort-list/description/)    
> 在 `$O(n\log n)$` 时间复杂度和常数级空间复杂度下，对链表进行排序。

链表排序问题可以参见[单链表排序 Sort List](https://devhui.com/2015/02/23/sort-list/)、[链表排序（冒泡、选择、插入、快排、归并、希尔、堆排序）](http://www.cnblogs.com/TenosDoIt/p/3666585.html)和[leetcode题解-链表排序算法 147. Insertion Sort List && 148 . Sort List](https://blog.csdn.net/liuchonge/article/details/74394995)。在链表中快速排序和归并排序的均摊复杂度都是`$O(n\log n)$`，空间复杂度也都是常数。

### 快速排序
快速排序的思路就是先将链表根据某个基础元素值分为两部分，比他大的和比他小的，然后一直这样分类下去，最后在进行合并即可。

```python

```


### 归并排序
首先看一下归并排序，其思路是先将所有元素两两划分为一组，然后递归合并，合并就相当于将两个排序好的链表合并为一个。这样最终我们就可以将链表排序好了。那么关键所在就是先分裂，在合并。

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def sortList(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        if not head or not head.next:
            return head
        # 使用两个指针，一快一慢，当快指针走到链表尾部时，慢指针正好到中间位置。
        fast = head
        slow = head
        prev = ListNode(-1)
        while fast and fast.next:
            # prev表示第一个链表的尾部
            prev = slow
            slow = slow.next
            fast = fast.next.next
        # 第一个链表的尾部断开链表，第二个链表的头部就是slow
        prev.next = None
        # 继续分裂
        l1 = self.sortList(head)
        l2 = self.sortList(slow)
        return self.merge(l1, l2)
    
    # 合并函数
    def merge(self, l1, l2):
        # 初始化有序链表的头
        l = ListNode(-1)
        p = l
        while l1 and l2:
            if l1.val < l2.val:
                p.next = l1
                l1 = l1.next
            else:
                p.next = l2
                l2 = l2.next
            p = p.next
        if l1:
            p.next = l1
        if l2:
            p.next = l2
        return l.next
```

# 相交链表问题
## 160. 相交链表
[相交链表](https://leetcode-cn.com/problems/intersection-of-two-linked-lists/description/)  
> 编写一个程序，找到两个单链表相交的起始节点。

把a、b链表弄成等长，然后一起遍历，最先相等的结点就是交点。
```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def getIntersectionNode(self, headA, headB):
        """
        :type head1, head1: ListNode
        :rtype: ListNode
        """
        # 把a、b链表弄成等长，然后一起遍历，最先相等的结点就是交点。
        a = headA
        b = headB
        if not a or not b:
            return None
        lena = 0  # 统计链表a的长度
        while a.next:
            a = a.next
            lena += 1
        lenb = 0  # 统计链表b的长度
        while b.next:
            b = b.next
            lenb += 1
        # 当两个链表尾部不相等，则没有相交
        if a != b:
            return None
        a = headA
        b = headB
        if lena > lenb:
            for i in range(lena-lenb):
                a = a.next
            while a and b:
                if a == b:
                    return a
                a = a.next
                b = b.next
        else:
            for i in range(lenb-lena):
                b = b.next
            while a and b:
                if a == b:
                    return a
                a = a.next
                b = b.next
```

# 环形链表问题
## 141. 环形链表
[环形链表](https://leetcode-cn.com/problems/linked-list-cycle/description/) 
> 给定一个链表，判断链表中是否有环。

定义两个指针，同时从链表的头节点出发，一个指针一次走一步，另一个指针一次走两步。如果走得快的指针追上了走得慢的指针，那么链表就是环形链表；如果走得快的指针走到了链表的末尾（next指向 NULL）都没有追上第一个指针，那么链表就不是环形链表。

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def hasCycle(self, head):
        """
        :type head: ListNode
        :rtype: bool
        """
        # 定义两个指针，同时从链表的头节点出发，一个指针一次走一步，另一个指针一次走两步。
        # 如果走得快的指针追上了走得慢的指针，那么链表就是环形链表；
        # 如果走得快的指针走到了链表的末尾（next指向 NULL）都没有追上第一个指针，那么链表就不是环形链表。
        if not head or not head.next:
            return False
        slow = head.next
        fast = slow.next
        while fast and slow:
            if fast == slow:
                return True
            slow = slow.next
            fast = fast.next
            if fast:
                fast = fast.next
        return False
```

## 142. 环形链表 II
[环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii/description/)
> 给定一个链表，返回链表开始入环的第一个节点。 如果链表无环，则返回 null。

解法参见[[Leetcode]142. Linked List Cycle II @python](https://blog.csdn.net/qian2729/article/details/50599772)   
依然是使用快慢指针，当快慢指针在相遇点相遇后，让慢指针回到head，faster留在相遇点。然后同时向前走，两个指针都是每次走一步，这样当慢指针走到环的入口时，快指针也会走到环的入口。

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def detectCycle(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        if not head or not head.next:
            return None
        fast = head
        slow = head
        # 找到第一次相遇的点
        while fast and fast.next:
            fast = fast.next.next
            slow = slow.next
            if fast == slow:
                break
        # 将slow移到链表头部，然后继续移动slow和fast指针，这次是一次移一步
        # 当再次相遇就是环的入口
        if slow == fast:
            slow = head
            while slow != fast:
                slow = slow.next
                fast = fast.next
            return slow
        return None
```

# 删除链表中的重复元素
## 83. 删除排序链表中的重复元素
[删除排序链表中的重复元素](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/description/)
> 给定一个排序链表，删除所有重复的元素，使得每个元素只出现一次。


```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def deleteDuplicates(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        if not head:
            return head
        # 初始化删除重复元素后的链表的头
        newhead = ListNode(-1)
        cur = head
        newhead.next = cur
        while cur:
            nextnode = cur.next
            # 循环找到第一个不相等的节点作为下一个节点
            while nextnode and nextnode.val == cur.val:
                nextnode = nextnode.next
            # 将当前节点的下一个节点的指针指向第一个不相等的节点
            cur.next = nextnode
            # 移动当前指针
            cur = cur.next
        return newhead.next
```

## 82. 删除排序链表中的重复元素 II
[删除排序链表中的重复元素 II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/description/)
> 给定一个排序链表，删除所有含有重复数字的节点，只保留原始链表中 没有重复出现 的数字。

参照[Leetcode 82:删除排序链表中的重复元素 II（最详细解决方案！！！）](https://blog.csdn.net/qq_17550379/article/details/80668036)

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def deleteDuplicates(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        newhead = ListNode(-1)
        newhead.next = head
        pre = newhead
        cur = head
        while cur:
            # 标记是否重复
            duplicate = False
            while cur.next and cur.val == cur.next.val:
                cur = cur.next
                duplicate = True
            if not duplicate:
                pre = cur
            else:
                pre.next = cur.next
            cur = cur.next
        return newhead.next
```

# 61. 旋转链表
[旋转链表](https://leetcode-cn.com/problems/rotate-list/description/)
> 给定一个链表，旋转链表，将链表每个节点向右移动 k 个位置，其中 k 是非负数。

参见[61.Rotate List 旋转链表](http://blog.leanote.com/post/liuliu5151@gmail.com/Rotate-List)
1. 设立两个指针fast, slow
2. fast（或者slow）扫一遍，拿到链表长度，则：步数 n =K%长度
3. fast 先走 n 步， 则fast 领先 slow 有n 步
4. fast和slow一起走，知道fast走到链表的最后一位
5. 更改指针，让fast.next = head; head = slow.next; slow.next = None

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def rotateRight(self, head, k):
        """
        :type head: ListNode
        :type k: int
        :rtype: ListNode
        """
        if not head:
            return head
        fast = head
        slow = head
        # 统计链表总长度
        count = 1
        while fast.next:
            fast = fast.next
            count += 1
        fast = head
        for i in range(k % count):
            fast = fast.next
        while fast.next:
            slow = slow.next
            fast = fast.next
        fast.next = head
        head = slow.next
        slow.next = None
        return head
```

# 参考文献
[面试精选：链表问题集锦](http://wuchong.me/blog/2014/03/25/interview-link-questions/)    
[单链表排序 Sort List](https://devhui.com/2015/02/23/sort-list/)    
[leetcode题解-链表排序算法 147. Insertion Sort List && 148 . Sort List](https://blog.csdn.net/liuchonge/article/details/74394995)