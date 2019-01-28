* [数据倾斜](#数据倾斜)
* [解决方式](#解决方式)
* [参考文献](#参考文献)

# 数据倾斜
简单的讲，数据倾斜就是我们在计算数据的时候，数据的分散度不够，导致大量的数据集中到了一台或者几台机器上计算，这些数据的计算速度远远低于平均计算速度，导致整个计算过程过慢。
# 解决方式
1. 增加reduce 的jvm内存
2. 增加reduce 个数
3. 自定义partitioner
4. 在 key 上面做文章，在 map 阶段将造成倾斜的key 先分成多组，例如 aaa 这个 key,map 时随机在 aaa 后面加上 1,2,3,4 这四个数字之一，把 key 先分成四组，先进行一次运算，之后再恢复 key 进行最终运算。
5. join 操作中，使用 map join 在 map 端就先进行 join ，免得到reduce 时卡住。

# 参考文献
[漫谈千亿级数据优化实践：数据倾斜（纯干货）](https://segmentfault.com/a/1190000009166436)   
[hive.groupby.skewindata环境变量与负载均衡](https://www.cnblogs.com/hankedang/p/5649489.html)     
[MapReduce如何解决数据倾斜？](https://www.zhihu.com/question/27593027)   
[浅析 Hadoop 中的数据倾斜](https://my.oschina.net/leejun2005/blog/100922)   
