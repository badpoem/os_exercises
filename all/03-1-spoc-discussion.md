# lec5 SPOC思考题


NOTICE
- 有"w3l1"标记的题是助教要提交到学堂在线上的。
- 有"w3l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

请简要分析最优匹配，最差匹配，最先匹配，buddy systemm分配算法的优势和劣势，并尝试提出一种更有效的连续内存分配算法 (w3l1)
```
  + 采分点：说明四种算法的优点和缺点
  - 答案没有涉及如下3点；（0分）
  - 正确描述了二种分配算法的优势和劣势（1分）
  - 正确描述了四种分配算法的优势和劣势（2分）
  - 除上述两点外，进一步描述了一种更有效的分配算法（3分）
 ```
- ①最优匹配 总是把既能满足需求又是最小的空闲分区进行分配，每次找到的第一个满足需求的空闲区必然是最优的，大部分分配的尺寸较小的时候效果很好，可以避免大的空闲分区被拆分同时减小了外部碎片的大小，而且算法相对简单。但是由于每次分配后剩余的空间一定最小，在存储器中留下许多难以利用的小空闲区，同时每次分配后必须重新排序而带来开销，而且释放分区较慢。
- ②最差匹配 总是从大的空闲区开始查寻，克服最优匹配留下许多小碎片的不足，中等大小分配较多时效果好，避免出现太多的小碎片。但是保留大的空闲区的可能性减小了，同时空闲区回收和最优匹配相同复杂，释放分区较慢。
- ③最先匹配 倾向于使用内存中低地址部分的空闲分区，从而保留高地址部分的大空闲区而为分配大的内存空间创造条件。但是低地址部分不断被划分而留下许多小空闲区难于利用，而每次查找从低地址部分开始增加了查找开销。
- ④buddy system 提高了分配的灵活性，适合于空闲块的合并，可以减少存储空间中的空洞，增加空间的利用率。但是实现相对较为复杂，而且释放存储空间时只归并互为伙伴的空闲块，还是会引入存储碎片，并且释放时块的回收过程是递归的，可能花费大量时间，另外分配与释放交替进行的时候可能导致对同一内存块不断分割和合并。
- ⑤新的连续内存分配算法：a.将内存空间划分成诺干固定块;b.用1个bit来表示1个block的空闲与分配状态;c.分配空间时查找空间管理的区间byte;d.free时则清空相应的bits，自动与原来的空闲bit连成一片完成回收
- ⑥或Solaris的Slab分配算法：涉及对象的状态，为每一种动态分配对象类分配对象缓存，当这些对象类需要内存分配时内核从他们各自的缓存中分配和释放对象，而对于其他对象Slab通过kmem_slab结构对其进行管理。

>  

## 小组思考题

请参考ucore lab2代码，采用`struct pmm_manager` 根据你的`学号 mod 4`的结果值，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在应用程序层面(可以 用python,ruby,C++，C，LISP等高语言)来实现，给出你的设思路，并给出测试用例。 (spoc)

-程序代码如下

'''
 #!/usr/bin/python
 #coding: utf-8

class PmmManager():
    def __init__(self, init_block):
        self.freed = [init_block]  # (size, start_address)
        self.used = []
    
    def near_block(self, block1, block2):
        if block1[1] < block2[1]:
            if block1[1] + block1[0] == block2[1]:
                return True
            else:
                return False
        else:
            if block2[1] + block2[0] == block1[1]:
                return True
            else:
                return False

    def merge_block(self, block1, block2):
        if block1[1] < block2[1]:
            return (block1[1], block1[0] + block2[0])
        else:
            return (block2[1], block1[0] + block2[0])

    def malloc(self, size):
        if len(self.freed) == 0 or self.freed[0][0] < size:
            return None                               # Not enough space
        else:
            block = self.freed[0]                     # Try the first block
            pointer = (size, block[1])                # Space to return
            self.freed.pop(0)
            self.used.append((block[1], size))
            self.freed.append((block[0] - size, block[1] + size))
            self.freed.sort(reverse=True)                   # Sort by size (big to small)
            return pointer 

    def free(self, block):
        if block not in self.used:
            return False                              # Cannot find the block
        else:
            self.used.remove(block)
            while True:
                merged = False
                for item in self.freed:
                    if near_block(item, block):
                        block = merge_block(item, block)
                        self.freed.remove(item)
                        merged = True
                if merged is False:
                    break
            self.freed.append(block)
            self.freed.sort(reverse=True)
            return True

def test_pmm():
    pmm = PmmManager((64, 0x00))
    space1 = pmm.malloc(16)
    print '[+] Malloc:' + str(space1)
    space2 = pmm.malloc(32)
    print '[+] Malloc:' + str(space2)
    space3 = pmm.malloc(32)
    print '[+] Malloc:' + str(space3)
    pmm.free(space2)
    print '[-] Free:' + str(space2)

if __name__ == "__main__":
    test_pmm()
'''

-  程序思路是使用2元组（块大小， 块起始地址）来表示一块内存空间，使用的分配方式为最差匹配，每次从freed表头部取出最大的那个块使用，切块之后分配过的空间放入used表中，然后每次分配结束对freed表进行从大到小排序即可。释放时不断检查释放块与freed表中各块是否相邻，然后合并即可。

--- 

## 扩展思考题

阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次
MMU的工作机理？

- [x]  

>  http://en.wikipedia.org/wiki/Memory_management_unit

L1和L2高速缓存有什么区别？

- [x]  

>  http://superuser.com/questions/196143/where-exactly-l1-l2-and-l3-caches-located-in-computer
>  Where exactly L1, L2 and L3 Caches located in computer?

>  http://en.wikipedia.org/wiki/CPU_cache
>  CPU cache

### 5.2 地址空间和地址生成
编译、链接和加载的过程了解？

- [x]  

>  

动态链接如何使用？

- [x]  

>  


### 5.3 连续内存分配
什么是内碎片、外碎片？

- [x]  

>  

为什么最先匹配会越用越慢？

- [x]  

>  

为什么最差匹配会的外碎片少？

- [x]  

>  

在几种算法中分区释放后的合并处理如何做？

- [x]  

>  

### 5.4 碎片整理
一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？

- [x]  

>  

### 5.5 伙伴系统
伙伴系统的空闲块如何组织？

- [x]  

>  

伙伴系统的内存分配流程？

- [x]  

>  

伙伴系统的内存回收流程？

- [x]  

>  

struct list_entry是如何把数据元素组织成链表的？

- [x]  

>  



