[toc]

LRU（Least recently used，**最近最少使用**）算法根据数据的**历史访问记录**来进行淘汰数据，其核心思想是“如果数据最近被访问过，那么将来被访问的几率也更高”。   

# Java实现
## 使用链表
最常见的实现是使用一个链表保存缓存数据，详细算法实现如下：    
![链表实现](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/lru%E9%93%BE%E8%A1%A8%E5%AE%9E%E7%8E%B0.png)    
1. 新数据插入到链表头部； 
2. 每当缓存命中（即缓存数据被访问），则将数据移到链表头部； 
3. 当链表满的时候，将链表尾部的数据丢弃。 
 

# 参考文献
[LRU算法的实现（Java版）](https://allenwind.github.io/2017/09/27/LRU%E7%AE%97%E6%B3%95%E7%9A%84%E5%AE%9E%E7%8E%B0%EF%BC%88Java%E7%89%88%EF%BC%89/)   
[LRU算法的实现（Python版）](https://allenwind.github.io/2017/09/17/LRU%E7%AE%97%E6%B3%95%E7%9A%84%E5%AE%9E%E7%8E%B0%EF%BC%88Python%E7%89%88%EF%BC%89/)   
[如何设计实现一个LRU Cache？](https://yikun.github.io/2015/04/03/%E5%A6%82%E4%BD%95%E8%AE%BE%E8%AE%A1%E5%AE%9E%E7%8E%B0%E4%B8%80%E4%B8%AALRU-Cache%EF%BC%9F/)   
[LRU缓存实现(Java)](https://www.cnblogs.com/lzrabbit/p/3734850.html)    
[缓存淘汰算法--LRU算法(java代码实现)](https://blog.csdn.net/wangxilong1991/article/details/70172302)