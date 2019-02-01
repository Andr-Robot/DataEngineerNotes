* [判断有向图是否有环及环中元素](#判断有向图是否有环及环中元素)

# 判断有向图是否有环及环中元素
**主要思路**： 
- dfs+栈：具体来说，遍历图中每个节点，若该节点还未被访问，则调用dfs。在访问节点n时，若该节点不在栈中，则将其入栈，否则说明存在环，并且环中元素为栈中从节点n到栈顶的所有点。

**dfs+栈实现**：
```python
# 有环则输出环中元素
def dfs(node, graph, visted, stack, res):
    # 当仅用于判断是否有环，可以在访问到第一个环的时候就停止访问 
    # if len(res) != 0:
    #     return
    if node in graph:
        visted[node] = True
        stack.append(node)
        for n in graph[node]:
            if not visted[n]:
                dfs(n, graph, visted, stack, res)
            else:
                # 输出环中的元素
                index = stack.index(n)
                res.append(stack[index:] + [n])
                break
        stack.pop(-1)

if __name__ == '__main__':
    graph = {}  # 存储图的结构，key是节点值，value是list格式的与该节点相连的节点值
    visted = {}  # 存储节点是否被访问，key是节点值，value是boolean类型
    stack = []  # 作为访问路径的栈
    n = int(input())  # 图中有n个边
    for i in range(n):
        n1, n2 = list(map(int, input().split()))
        if n1 not in graph:
            graph[n1] = [n2]
        elif n2 not in graph[n1]:
            graph[n1].append(n2)
        if n1 not in visted:
            visted[n1] = False
        if n2 not in visted:
            visted[n2] = False
    res = []
    for node in visted.keys():
        if not visted[node]:
            dfs(node, graph, visted, stack, res)
    if len(res) > 0:
        print('有环')
        print(res)
    else:
        print('无环')
    
输入：
8
1 2
2 3
3 1
3 4
4 5
5 6
6 7
7 5

输出：
有环
[[1, 2, 3, 1], [5, 6, 7, 5]]
```
 
- 拓扑排序：方法是重复寻找一个入度为0的顶点，将该顶点从图中删除（即放进一个队列里存着，这个队列的顺序就是最后的拓扑排序，具体见程序），并将该结点及其所有的出边从图中删除（即该结点指向的结点的入度减1），最终若图中全为入度为1的点，则这些点至少组成一个回路。     
    参见[有向无环图的拓扑排序](https://www.cnblogs.com/en-heng/p/5085690.html)   
    算法流程：
    1. 计算图中所有点的入度，把入度为0的点加入栈
    2. 如果栈非空：
        - 取出栈顶顶点a，输出该顶点值，删除该顶点
        - 从图中删除所有以a为起始点的边，如果删除的边的另一个顶点入度为0，则把它入栈
    3. 如果图中还存在顶点，则表示图中存在环；否则输出的顶点就是一个拓扑排序序列

**拓扑排序实现：**

```python
def topsort(G):
    in_degrees = dict((u, 0) for u in G)  # 计算入度
    for u in G:
        for v in G[u]:
            in_degrees[v] += 1  # 每一个节点的入度
    node = list(G.keys())  # 存储当前图中的所有节点
    Q = [u for u in G if in_degrees[u] == 0]  # 入度为 0 的节点
    S = []
    while Q:
        u = Q.pop()  # 默认从最后一个移除
        S.append(u)
        node.remove(u)  # 从图删除该节点
        for v in G[u]:
            in_degrees[v] -= 1   # 并移除其指向
            if in_degrees[v] == 0:
                Q.append(v)
    if len(node) == 0:
        print('无环')
        return S
    else:
        print('有环')
        return -1

if __name__ == '__main__':
    G = {
        'a': 'bf',
        'b': 'cdf',
        'c': 'd',
        'd': 'ef',
        'e': 'f',
        'f': ''
    }
    print(topsort(G))

输出：
无环
['a', 'b', 'c', 'd', 'e', 'f']
```
