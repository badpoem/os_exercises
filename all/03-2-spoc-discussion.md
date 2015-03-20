# lec6 SPOC思考题


NOTICE
- 有"w3l2"标记的题是助教要提交到学堂在线上的。
- 有"w3l2"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

（1） (w3l2) 请简要分析64bit CPU体系结构下的分页机制是如何实现的
```
  + 采分点：说明64bit CPU架构的分页机制的大致特点和页表执行过程
  - 答案没有涉及如下3点；（0分）
  - 正确描述了64bit CPU支持的物理内存大小限制（1分）
  - 正确描述了64bit CPU下的多级页表的级数和多级页表的结构或反置页表的结构（2分）
  - 除上述两点外，进一步描述了在多级页表或反置页表下的虚拟地址-->物理地址的映射过程（3分）
 ```
- ①32位的Intel CPU可以支持4G物理内存，而打开PAE后可以支持64G。对于64位CPU而言理论上最高可以支持16EB，但实际上这么大的地址空间对于目前形势是一种浪费，所以实际支持物理内存并没有那么大。比如AMD64架构支持4PB52位地址总线，而64位的Linux支持46位4TB的物理地址空间。
- ②现通行的64位CPU体系结构下的页表一般是分为3级或4级的，当然更多的是引入了4级页表。考虑到它们工作原理相同，所以介绍3级页表。1级页表PTE大小为32位，虚拟地址结构为页号+页内地址；二级页表，虚拟地址组成为目录地址+页表地址+页内偏移；对32位系统而言加入物理地址扩展PAE后二级页表无法满足要求，故引入三级页表，增加PMD中间目录一级。
- ③多级页表结构为Global Dir+Upper Dir+Middle Dir+Table+Offeset，到物理地址的映射过程为首先从cr3寄存器中找到PGD的首地址，逐步进行计算先后得到页全局目录、页上级目录、页中间目录、页表、页，得到物理地址。
- ④虽然在IA-32 IA-64等系统中一般使用四级页表处理64位操作系统，但是Power PC等体系中使用倒排页表，也即反置页表来解决这个问题，使用页框号而不是虚拟页号来索引页表项，但是这并不是X86体系中常用的方法。倒排页表由页号+进程ID+控制位+链指针构成。
- ⑤倒排页表的虚拟地址部分由页号+偏移量组成，页号使用一个简单的散列函数映射到散列表中，散列表包含一个指向倒排表的指针，倒排表中含有页表项，通过这个结构散列表和倒排表中各有一项对应于一个实存页。虚拟地址页号经由散列函数和倒排表映射后得到页框号，与偏移量共同组成物理地址。

>  

## 小组思考题
---

（1）(spoc) 某系统使用请求分页存储管理，若页在内存中，满足一个内存请求需要150ns。若缺页率是10%，为使有效访问时间达到0.5ms,求不在内存的页面的平均访问时间。请给出计算步骤。 

- 解：设不在内存的页面的平均访问时间为x us(陈老师课上表示修改为微秒)
- 则由题有 150\*(1-10%)+x\*10%=500
- 解得 x=3650
- 故不在内存的页面的平均访问时间为3650us 合3.65ms

> 500=0.9\*150+0.1\*x

（2）(spoc) 有一台假想的计算机，页大小（page size）为32 Bytes，支持32KB的虚拟地址空间（virtual address space）,有4KB的物理内存空间（physical memory），采用二级页表，一个页目录项（page directory entry ，PDE）大小为1 Byte,一个页表项（page-table entries
PTEs）大小为1 Byte，1个页目录表大小为32 Bytes，1个页表大小为32 Bytes。页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐）。

PTE格式（8 bit） :
```
  VALID | PFN6 ... PFN0
```
PDE格式（8 bit） :
```
  VALID | PT6 ... PT0
```
其
```
VALID==1表示，表示映射存在；VALID==0表示，表示映射不存在。
PFN6..0:页帧号
PT6..0:页表的物理基址>>5
```
在[物理内存模拟数据文件](./03-2-spoc-testdata.md)中，给出了4KB物理内存空间的值，请回答下列虚地址是否有合法对应的物理内存，请给出对应的pde index, pde contents, pte index, pte contents。
```
Virtual Address 6c74
Virtual Address 6b22
Virtual Address 03df
Virtual Address 69dc
Virtual Address 317a
Virtual Address 4546
Virtual Address 2c03
Virtual Address 7fd7
Virtual Address 390e
Virtual Address 748b
```

比如答案可以如下表示：
```
Virtual Address 7570:
  --> pde index:0x1d  pde contents:(valid 1, pfn 0x33)
    --> pte index:0xb  pte contents:(valid 0, pfn 0x7f)
      --> Fault (page table entry not valid)
      
Virtual Address 21e1:
  --> pde index:0x8  pde contents:(valid 0, pfn 0x7f)
      --> Fault (page directory entry not valid)

Virtual Address 7268:
  --> pde index:0x1c  pde contents:(valid 1, pfn 0x5e)
    --> pte index:0x13  pte contents:(valid 1, pfn 0x65)
      --> Translates to Physical Address 0xca8 --> Value: 16
```

- 使用一个python的脚本来完成工作，首先从testdata文件中dump出内存数据，然后根据输入的虚拟地址进行转换即可。

```
    #!/usr/bin/python
    # coding: utf-8

    pdbr = 0x220
    valid_mask = 0x80
    valid_shift = 7
    pde_mask = 0x7c00
    pde_shift = 10
    pte_mask = 0x03e0
    pte_shift = 5
    pfn_mask = 0x007f

    offset_mask = 0x001f
    page_mask = 0x0fe0

    memory = []    # Memory data

    def read_byte(memaddr):
        return memory[(memaddr & page_mask) >> 5][memaddr & offset_mask]

    def dump_memory():
        filename = '03-2-spoc-testdata.md'
        lines = range(5, 133)
        data = open(filename, 'r').readlines()
        for i in lines:
            l = data[i][8:].strip().split(' ')
            memory.append([int(x, 16) for x in l])

    def mmu(va):
        print 'Virtual Address 0x%04x' % va
        
        # first find pde
        pde_index = (va & pde_mask) >> pde_shift
        pde_entry = read_byte(pdbr + pde_index)
        valid = (pde_entry & valid_mask) >> valid_shift
        pfn = pde_entry & pfn_mask
        print '  --> pde index:0x%02x pde contents:(valid %d, pfn 0x%02x)' % (pde_index, valid, pfn)

        if valid == 0:
            print '    --> Fault (page directory entry not valid)'
            return

        # then find pte
        pte_index = (va & pte_mask) >> pte_shift
        pte_entry = memory[pfn][pte_index] 
        valid = (pte_entry & valid_mask) >> valid_shift
        pfn = pte_entry & pfn_mask
        print '    --> pte index:0x%02x pte contents:(valid %d, pfn 0x%02x)' % (pte_index, valid, pfn)

        if valid == 0:
            print '      --> Fault (page table entry not valid)'
            return

        # at last get the physical value

        ph_addr = (pfn << 5) + (va & offset_mask)
        print '      --> Translates to Physical Address 0x%03x --> Value: 0x%02x' % (ph_addr, read_byte(ph_addr))


    if __name__ == '__main__':
        dump_memory()
        vas = [0x6c74, 0x6b22, 0x03df, 0x69dc, 0x317a, 0x4546, 0x2c03, 0x7fd7, 0x390e, 0x748b]
        for va in vas:
            mmu(va)
```

- 运行脚本得到输出

```
Virtual Address 0x6c74
  --> pde index:0x1b pde contents:(valid 1, pfn 0x20)
    --> pte index:0x03 pte contents:(valid 1, pfn 0x61)
      --> Translates to Physical Address 0xc34 --> Value: 0x06
Virtual Address 0x6b22
  --> pde index:0x1a pde contents:(valid 1, pfn 0x52)
    --> pte index:0x19 pte contents:(valid 1, pfn 0x47)
      --> Translates to Physical Address 0x8e2 --> Value: 0x1a
Virtual Address 0x03df
  --> pde index:0x00 pde contents:(valid 1, pfn 0x5a)
    --> pte index:0x1e pte contents:(valid 1, pfn 0x05)
      --> Translates to Physical Address 0x0bf --> Value: 0x0f
Virtual Address 0x69dc
  --> pde index:0x1a pde contents:(valid 1, pfn 0x52)
    --> pte index:0x0e pte contents:(valid 0, pfn 0x7f)
      --> Fault (page table entry not valid)
Virtual Address 0x317a
  --> pde index:0x0c pde contents:(valid 1, pfn 0x18)
    --> pte index:0x0b pte contents:(valid 1, pfn 0x35)
      --> Translates to Physical Address 0x6ba --> Value: 0x1e
Virtual Address 0x4546
  --> pde index:0x11 pde contents:(valid 1, pfn 0x21)
    --> pte index:0x0a pte contents:(valid 0, pfn 0x7f)
      --> Fault (page table entry not valid)
Virtual Address 0x2c03
  --> pde index:0x0b pde contents:(valid 1, pfn 0x44)
    --> pte index:0x00 pte contents:(valid 1, pfn 0x57)
      --> Translates to Physical Address 0xae3 --> Value: 0x16
Virtual Address 0x7fd7
  --> pde index:0x1f pde contents:(valid 1, pfn 0x12)
    --> pte index:0x1e pte contents:(valid 0, pfn 0x7f)
      --> Fault (page table entry not valid)
Virtual Address 0x390e
  --> pde index:0x0e pde contents:(valid 0, pfn 0x7f)
    --> Fault (page directory entry not valid)
Virtual Address 0x748b
  --> pde index:0x1d pde contents:(valid 1, pfn 0x00)
    --> pte index:0x04 pte contents:(valid 0, pfn 0x7f)
      --> Fault (page table entry not valid)
```


（3）请基于你对原理课二级页表的理解，并参考Lab2建页表的过程，设计一个应用程序（可基于python, ruby, C, C++，LISP等）可模拟实现(2)题中描述的抽象OS，可正确完成二级页表转换。


（4）假设你有一台支持[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)的机器，请问你如何设计操作系统支持这种类型计算机？请给出设计方案。

 (5)[X86的页面结构](http://os.cs.tsinghua.edu.cn/oscourse/OS2015/lecture06#head-1f58ea81c046bd27b196ea2c366d0a2063b304ab)
--- 

## 扩展思考题

阅读64bit IBM Powerpc CPU架构是如何实现[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)，给出分析报告。

--- 
