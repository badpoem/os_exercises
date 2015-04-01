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

```
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
```

```
证明：
        原理:因为小的物理页帧的栈包含于大数目的物理页帧的栈
        证明根据课堂老师给出的基础：
            来证明s(t) 始终包含于 s'(t)
            利用归纳法，假设 1<=i<=t-1 时 s(i)包含于s'(i)，现在要证s(t)依然包含于s'(t)
            (1) b(t)同时属于s(t)和s'(t)：此时s(t)和s'(t)都不发生变化，满足包含关系；
            (2) b(t)不属于s(t),属于s'(t)：s(t) 替换后，由于b(t)∈s(t)，所以s(t)包含于s'(t)
            (3)  (1)和(2)很容易证明，
                对于b(t)同时不属于s(t-1)和s'(t-1)的情况，我们依然按照视频里栈的方式对s(t-1)和s'(t-1)排序
                由于s(t-1)包含于s'(t-1),所以s(t-1)内每一个元素都存在于s'(t-1)中。
                现在两个栈都是按最后一次访问的时间的顺序来排列的，由于s(t)在进行替换时会替换s(t-1)里面最长时间没被访问的元素(栈底)，设为a,那么a显然也存在于s'(t-1)里面，并且它不一定是s'(t-1)的栈底。
                    A. 当a是s'(t-1)的栈底时，s(t)和s'(t)替换的都是a, s(t) = s(t-1) - {a} + {b(t)} , s'(t) = s'(t-1) - {a} + {b(t)}
                    B. 当a不是s'(t-1)的栈底是，则s'(t-1)的栈底c必然不属于s(t-1)，否则就会与a是s(t-1)的栈底矛盾（即c比a有更长的时间未被访问），此种情况下s(t)和s'(t)依然满足包含关系
            (4) 由归纳假设可以得知此种情况不存在
 ```

(2)（spoc）根据你的`学号 mod 4`的结果值，确定选择四种替换算法（0：LRU置换算法，1:改进的clock 页置换算法，2：工作集页置换算法，3：缺页率置换算法）中的一种来设计一个应用程序（可基于python, ruby, C, C++，LISP等）模拟实现，并给出测试。请参考如python代码或独自实现。
 - [页置换算法实现的参考实例](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab3/page-replacement-policy.py)

代码如下

```
#!/usr/bin/env python
# coding: utf-8

class Page():
    
    def __init__(self, index):
        self.ref = 0     # reference
        self.mod = 0     # modify
        self.idx = index


def test_clock(memory_size=8, access_list=[]):

    in_page = []
    memory = {}
    head = 0
    pattern = [(0,0), (0,1), (1,0), (1,1)]
    # modify pattern
    change_pattern = {(0,0) : (0,0), (0,1) : (0,0), (1,0) : (0,0), (1,1) : (0,1)}
    miss = 0

    for i in range(memory_size):
        in_page.append(Page(i))
        memory[i] = i
    
    #Start resolve
    for access in access_list:
        print 'access :' + str(access)
        if access[0] in memory.keys():
            # page in memory
            if access[1] == 'r':
                in_page[memory[access[0]]].ref = 1
            else:
                in_page[memory[access[0]]].mod = 1
        else:
            # page not in memory
            print 'page fault: head=%d' % head
            miss += 1
            while True:
                p = (in_page[head].ref, in_page[head].mod)
                if p == (0,0):
                    break
                else:
                    # give it a second chance
                    p = (in_page[head].ref, in_page[head].mod)
                    p = change_pattern[p]
                    in_page[head].ref = p[0]
                    in_page[head].mod = p[1]
                head = (head + 1) % memory_size

            # swap the victim
            idx = in_page[head].idx
            memory.pop(idx)
            print 'victim is %d' % idx
            memory[access[0]] = head
            if access[1] == 'r':
                in_page[head].ref = 1
                in_page[head].mod = 0
            else:
                in_page[head].ref = 0
                in_page[head].mod = 1
    return miss

if __name__ == '__main__':
    memory_size = 8
    access_list = [(0,'r'), (1,'r'), (3, 'w'), (10, 'w'), (2, 'r'), (5, 'w'), (9, 'r'), (0, 'w'), (11, 'r')]
    miss = test_clock(memory_size, access_list)
    print 'miss=%d' % miss 
```

## 扩展思考题
（1）了解LIRS页置换算法的设计思路，尝试用高级语言实现其基本思路。此算法是江松博士（导师：张晓东博士）设计完成的，非常不错！

参考信息：

 - [LIRS conf paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang02_LIRS.pdf)
 - [LIRS journal paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang05_LIRS.pdf)
 - [LIRS-replacement ppt1](http://dragonstar.ict.ac.cn/course_09/XD_Zhang/(6)-LIRS-replacement.pdf)
 - [LIRS-replacement ppt2](http://www.ece.eng.wayne.edu/~sjiang/Projects/LIRS/sig02.ppt)
