#lec9 虚存置换算法spoc练习

## 个人思考题
1. 置换算法的功能？

2. 全局和局部置换算法的不同？

3. 最优算法、先进先出算法和LRU算法的思路？

4. 时钟置换算法的思路？

5. LFU算法的思路？

6. 什么是Belady现象？

7. 几种局部置换算法的相关性：什么地方是相似的？什么地方是不同的？为什么有这种相似或不同？

8. 什么是工作集？

9. 什么是常驻集？

10. 工作集算法的思路？

11. 缺页率算法的思路？

12. 什么是虚拟内存管理的抖动现象？

13. 操作系统负载控制的最佳状态是什么状态？

## 小组思考题目

----
(1)（spoc）请证明为何LRU算法不会出现belady现象

虚拟页数量为M 访问页的序列为b(t) 0<b(t)<M-1 在时刻t 物理页帧集合为S(t) 元素范围[0,N] 物理页帧的集合大小为N N<M
如果 b(t)∈S(t)则命中 S(t+1)=S(t)
如果不属于 则表示缺页 此时有一个块换出v(t) S(t+1)=(S(t)-v(t))∪b(t)

S(1)=1,2,...N, S'(1)=1,2,...N, 证明LRU算法在S'大小为N+k时候缺失率低于S
反证法
分三种情况讨论
1. b(t)属于S(t) 且属于S'(t)
2. 不属于 属于
3. 不属于 不属于
4. 属于 不属于

1. 若b(t)∈S(t)，且b(t)∈S'(t)
  则S与S'均命中，S S'保持不变，不会触发缺页异常。那么S与S'的缺页次数保持不变。且如果S(t)包含于S'(t)，则S(t+1)包含于S'(t+1)
2. 若不属于 属于 则S缺失，触发缺页异常，缺页次数+1，而S'命中，不触发缺页异常，缺页次数保持不变。且如果S(t)包含于S'(t)，则由于b(t)属于S'(t)，则S(t+1)包含于S'(t+1)
3. 若不属于 不属于 则S与S'均缺失，触发缺页异常，缺页次数+1.则如果S(t)包含于S'(t)，可以分为两种情况讨论，第一是v(t)==v'(t)，则S(t+1)包含于S'(t+1)；第二是v(t)!=v'(t)，则v(t)是S中近期最少使用的项，若v'(t)属于S则与v(t)近期最少矛盾，所以v'(t)一定属于S'但是不属于S。所以仍然有S(t+1)包含于S'(t+1)
4. 由于开始有S(1)=S'(1)，且除了第四种以外的所有推演均基于以上三种模式，则始终有S包含于S'，而不可能出现第四种情况，b(t)属于S(t)却不属于S'(t)，即S(t)不包含于S'(t)，所以第四种情况不可能出现。
所以由1 2 3 归纳得到S缺页次数总是大于等于S'，所以得证。

(2)（spoc）根据你的`学号 mod 4`的结果值，确定选择四种替换算法（0：LRU置换算法，1:改进的clock 页置换算法，2：工作集页置换算法，3：缺页率置换算法）中的一种来设计一个应用程序（可基于python, ruby, C, C++，LISP等）模拟实现，并给出测试。请参考如python代码或独自实现。
 - [页置换算法实现的参考实例](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab3/page-replacement-policy.py)
 
## 扩展思考题
（1）了解LIRS页置换算法的设计思路，尝试用高级语言实现其基本思路。此算法是江松博士（导师：张晓东博士）设计完成的，非常不错！

参考信息：

 - [LIRS conf paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang02_LIRS.pdf)
 - [LIRS journal paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang05_LIRS.pdf)
 - [LIRS-replacement ppt1](http://dragonstar.ict.ac.cn/course_09/XD_Zhang/(6)-LIRS-replacement.pdf)
 - [LIRS-replacement ppt2](http://www.ece.eng.wayne.edu/~sjiang/Projects/LIRS/sig02.ppt)
