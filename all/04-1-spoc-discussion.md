#lec8 虚拟内存spoc练习


NOTICE
- 有"w4l2"标记的题是助教要提交到学堂在线上的。
- 有"w4l2"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题

### 内存访问局部性的应用程序例子
---
(1)(w4l2)下面是一个体现内存访问局部性好的简单应用程序例子，请参考，在linux中写一个简单应用程序，体现内存局部性差，并给出其执行时间。
```
#include <stdio.h>
#define NUM 1024
#define COUNT 10
int A[NUM][NUM];
void main (void) {
  int i,j,k;
  for (k = 0; k<COUNT; k++)
  for (i = 0; i < NUM; i++)
  for (j = 0; j	 < NUM; j++)
      A[i][j] = i+j;
  printf("%d count computing over!\n",i*j*k);
}
```
可以用下的命令来编译和运行此程序：
```
gcc -O0 -o goodlocality goodlocality.c
time ./goodlocality
```
可以看到其执行时间。

- 可以直接将上一个程序中的A[i][j] = i+j; 改为A[j][i] = i+j; 就可以写成一个局部性差的例子。用time测试的结果为

```
===========Good Locality============
10485760 count computing over!

real	0m0.082s
user	0m0.074s
sys	0m0.004s
===========Bad Locality=============
10485760 count computing over!

real	0m0.606s
user	0m0.563s
sys	0m0.007s
```

- 从上面的结果可以看出局部性差的结果606ms比局部性好的结果82ms的时间差了数倍。这是由于c程序的数组是行存储的，一次会将一行调入cache中读写速度比较快，而如果每次都读取不同行的数据则需要不断访问主存运行时间就会增长。

## 小组思考题目
----

### 缺页异常嵌套

（1）缺页异常可用于虚拟内存管理中。如果在中断服务例程中进行缺页异常的处理时，再次出现缺页异常，这时计算机系统（软件或硬件）会如何处理？请给出你的合理设计和解释。

### 缺页中断次数计算
（2）如果80386机器的一条机器指令(指字长4个字节)，其功能是把一个32位字的数据装入寄存器，指令本身包含了要装入的字所在的32位地址。这个过程最多会引起几次缺页中断？
> 提示：内存中的指令和数据的地址需要考虑地址对齐和不对齐两种情况。需要考虑页目录表项invalid、页表项invalid、TLB缺失等是否会产生中断？

### 虚拟页式存储的地址转换

（3）(spoc) 有一台假想的计算机，页大小（page size）为32 Bytes，支持8KB的虚拟地址空间（virtual address space）,有4KB的物理内存空间（physical memory），采用二级页表，一个页目录项（page directory entry ，PDE）大小为1 Byte,一个页表项（page-table entries
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
VALID==1表示，表示映射存在；VALID==0表示，表示内存映射不存在（有两种情况：a.对应的物理页帧swap out在硬盘上；b.既没有在内存中，页没有在硬盘上）。
PFN6..0:页帧号或外存中的后备页号
PT6..0:页表的物理基址>>5
```

已经建立好了1个页目录表和8个页表，且页目录表的index为0~7的页目录项分别对应了这8个页表。

在[物理内存模拟数据文件](./04-1-spoc-memdiskdata.md)中，给出了4KB物理内存空间和4KBdisk空间的值，PDBR的值。

请回答下列虚地址是否有合法对应的物理内存，请给出对应的pde index, pde contents, pte index, pte contents，the value of addr in phy page OR disk sector。
```
Virtual Address 6653:
Virtual Address 1c13:
Virtual Address 6890:
Virtual Address 0af6:
Virtual Address 1e6f:
```

**提示:**
```
页大小（page size）为32 Bytes(2^5)
页表项1B

8KB的虚拟地址空间(2^15)
一级页表：2^5
PDBR content: 0xd80（1101_100 0_0000, page 0x6c）

page 6c: e1(1110 0001) b5(1011 0101) a1(1010 0001) c1(1100 0001)
         b3(1011 0011) e4(1110 0100) a6(1010 0110) bd(1011 1101)
二级页表：2^5
页内偏移：2^5

4KB的物理内存空间（physical memory）(2^12)
物理帧号：2^7

Virtual Address 0330(0 00000 11001 1_0000):
  --> pde index:0x0(00000)  pde contents:(0xe1, 11100001, valid 1, pfn 0x61(page 0x61))
  page 61: 7c 7f 7f 4e 4a 7f 3b 5a 2a be 7f 6d 7f 66 7f a7
           69 96 7f c8 3a 7f a5 83 07 e3 7f 37 62 30 7f 3f 
    --> pte index:0x19(11001)  pte contents:(0xe3, 1 110_0011, valid 1, pfn 0x63)
  page 63: 16 00 0d 15 00 1c 1d 16 02 02 0b 00 0a 00 1e 19
           02 1b 06 06 14 1d 03 00 0b 00 12 1a 05 03 0a 1d
      --> To Physical Address 0xc70(110001110000, 0xc70) --> Value: 02

Virtual Address 1e6f(0 001_11 10_011 0_1111):
  --> pde index:0x7(00111)  pde contents:(0xbd, 10111101, valid 1, pfn 0x3d)
  page 6c: e1 b5 a1 c1 b3 e4 a6 bd 7f 7f 7f 7f 7f 7f 7f 7f
           7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f
  page 3d: f6 7f 5d 4d 7f 04 29 7f 1e 7f ef 51 0c 1c 7f 7f
           7f 76 d1 16 7f 17 ab 55 9a 65 ba 7f 7f 0b 7f 7f 
    --> pte index:0x13  pte contents:(0x16, valid 0, pfn 0x16)
  disk 16: 00 0a 15 1a 03 00 09 13 1c 0a 18 03 13 07 17 1c 
           0d 15 0a 1a 0c 12 1e 11 0e 02 1d 10 15 14 07 13
      --> To Disk Sector Address 0x2cf(0001011001111) --> Value: 1c
```

- 程序如下

```
    #!/usr/bin/env python
    # coding: utf-8

    __author__ = 'Pei Zhongyu'

    pdbr = 0xd80
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
    disk = []      # Disk data

    mem_lines = range(5, 133)
    disk_lines = range(137, 265)

    def print_page(pageaddr, title='page', target=memory):
        print '  ' + title + ' %02x:' % pageaddr,
        for i in target[pageaddr]:
            print '%02x' % i,
        print

    def read_byte(memaddr, target=memory):
        return target[(memaddr & page_mask) >> 5][memaddr & offset_mask]

    def dump_memory(filename='04-1-spoc-memdiskdata.md', lines=mem_lines, target=memory):
        data = open(filename, 'r').readlines()
        for i in lines:
            l = data[i][8:].strip().split(' ')
            target.append([int(x, 16) for x in l])

    def mmu(va):
        print
        print 'Virtual Address 0x%04x' % va
        
        # first find pde
        pde_index = (va & pde_mask) >> pde_shift
        pde_entry = read_byte(pdbr + pde_index)
        valid = (pde_entry & valid_mask) >> valid_shift
        pfn = pde_entry & pfn_mask
        print '  --> pde index:0x%02x pde contents:(valid %d, pfn 0x%02x)' % (pde_index, valid, pfn)

        if valid == 0 or pde_index > 7:
            print '    --> Fault (page directory entry not valid)'
            return
        print_page(pfn) 
        
        # then find pte
        pte_index = (va & pte_mask) >> pte_shift
        pte_entry = memory[pfn][pte_index] 
        valid = (pte_entry & valid_mask) >> valid_shift
        pfn = pte_entry & pfn_mask
        print '    --> pte index:0x%02x pte contents:(valid %d, pfn 0x%02x)' % (pte_index, valid, pfn)

        # switch to disk
        if valid == 0:
            disk_addr = (pfn << 5) + (va & offset_mask)
            print_page(pfn, 'disk', disk)
            print '      --> To Disk Sector Address 0x%03x --> Value: 0x%02x' % (disk_addr, read_byte(disk_addr, disk))
        else:
            # at last get the physical value
            ph_addr = (pfn << 5) + (va & offset_mask)
            print_page(pfn)
            print '      --> To Physical Address 0x%03x --> Value: 0x%02x' % (ph_addr, read_byte(ph_addr))

    if __name__ == '__main__':
        dump_memory(lines=mem_lines, target=memory)
        dump_memory(lines=disk_lines, target=disk)
        test_vas = [0x0330, 0x1e6f]
        vas = [0x6653, 0x1c13, 0x6890, 0x0af6, 0x1e6f]
        for va in vas:
            mmu(va)
```

- 得到的输出为

```
Virtual Address 0x6653
  --> pde index:0x19 pde contents:(valid 0, pfn 0x7f)
    --> Fault (page directory entry not valid)

Virtual Address 0x1c13
  --> pde index:0x07 pde contents:(valid 1, pfn 0x3d)
  page 3d: f6 7f 5d 4d 7f 04 29 7f 1e 7f ef 51 0c 1c 7f 7f 7f 76 d1 16 7f 17 ab 55 9a 65 ba 7f 7f 0b 7f 7f
    --> pte index:0x00 pte contents:(valid 1, pfn 0x76)
  page 76: 1a 1b 1c 10 0c 15 08 19 1a 1b 12 1d 11 0d 14 1e 1c 18 02 12 0f 13 1a 07 16 03 06 18 0a 19 03 04
      --> To Physical Address 0xed3 --> Value: 0x12

Virtual Address 0x6890
  --> pde index:0x1a pde contents:(valid 0, pfn 0x7f)
    --> Fault (page directory entry not valid)

Virtual Address 0x0af6
  --> pde index:0x02 pde contents:(valid 1, pfn 0x21)
  page 21: 7f 7f 36 8e 7f 33 d5 82 7f 7f 79 2b 7f 7f 7f 7f 7f f1 7f 7f 71 7f 7f 7f 63 7f 2f dd 67 7f f9 32
    --> pte index:0x17 pte contents:(valid 0, pfn 0x7f)
  disk 7f: 07 1a 19 1d 15 0f 1d 01 1c 17 06 09 1a 03 08 04 06 04 0a 1b 12 04 03 1e 07 08 05 08 1d 0b 0c 0a
      --> To Disk Sector Address 0xff6 --> Value: 0x03

Virtual Address 0x1e6f
  --> pde index:0x07 pde contents:(valid 1, pfn 0x3d)
  page 3d: f6 7f 5d 4d 7f 04 29 7f 1e 7f ef 51 0c 1c 7f 7f 7f 76 d1 16 7f 17 ab 55 9a 65 ba 7f 7f 0b 7f 7f
    --> pte index:0x13 pte contents:(valid 0, pfn 0x16)
  disk 16: 00 0a 15 1a 03 00 09 13 1c 0a 18 03 13 07 17 1c 0d 15 0a 1a 0c 12 1e 11 0e 02 1d 10 15 14 07 13
      --> To Disk Sector Address 0x2cf --> Value: 0x1c
```

## 扩展思考题
---
(1)请分析原理课的缺页异常的处理流程与lab3中的缺页异常的处理流程（分析粒度到函数级别）的异同之处。
