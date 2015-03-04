# lab0 SPOC思考题

## 个人思考题

---

能否读懂ucore中的AT&T格式的X86-32汇编语言？请列出你不理解的汇编语言。
- 能够
- 原本不能够的也已通过google解决

>  http://www.imada.sdu.dk/Courses/DM18/Litteratur/IntelnATT.htm

虽然学过计算机原理和x86汇编（根据THU-CS的课程设置），但对ucore中涉及的哪些硬件设计或功能细节不够了解？
- （就现在通过课程了解到的而言有）中断的具体实现是什么样的
- 用户态和核心态之间的转换是怎么实现的
- 文件系统是如何组织和管理文件的

>   


哪些困难（请分优先级）会阻碍你自主完成lab实验？
- 对于课程所学不够了解，不能很好地将知识与代码结合。
- （应该只有这样的困难才会阻碍自助完成而选择向老师或同学求助吧 感觉客观原因应该不会有阻碍）

>   

如何把一个在gdb中或执行过程中出现的物理/线性地址与你写的代码源码位置对应起来？
- 使用gdb调试、查看代码堆栈

>   

了解函数调用栈对lab实验有何帮助？
- lab实验中会有很多地方使用函数调用栈 可以更好地调试代码，理解程序的运行机制 减少问题的出现并且可以更好地解决出现的问题

>   

你希望从lab中学到什么知识？
- 与操作系统中学习到的书本知识相结合
- 如何管理和维护较为复杂的程序
- 加强自己阅读代码和coding的能力
- 培养全局观

>   

---

## 小组讨论题

---

搭建好实验环境，请描述碰到的困难和解决的过程。
- 使用百度云下载时候因为文件过大，居然要求安装百度云管家

> 

熟悉基本的git命令行操作命令，从github上的[ucore git repo](http://www.github.com/chyyuu/ucore_lab)下载ucore lab实验
- 使用git命令完成os_exercises的保存、复制、更新
- 已下载

> 

尝试用qemu+gdb（or ECLIPSE-CDT）调试lab1
- 已尝试

> 

对于如下的代码段，请说明”：“后面的数字是什么含义
```
/* Gate descriptors for interrupts and traps */
struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
};
```
- 为对应变量位域的大小，即表示对应变量所占位数

> 

对于如下的代码段，
```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 0,1,2,3);
```
请问执行上述指令后， intr的值是多少？
- 65538

> 

请分析 [list.h](https://github.com/chyyuu/ucore_lab/blob/master/labcodes/lab2/libs/list.h)内容中大致的含义，并能include这个文件，利用其结构和功能编写一个数据结构链表操作的小C程序
- 声明了结构体：list的entry，即链表的条目。定义了链表的创建、添加、删除、清空等操作。
- #include "list.h"
- int main(){
- list_entry* en = (list_entry)malloc(sizeof(list_entry));
- list_init(en);
- return 0;}

> 

---

## 开放思考题

---

是否愿意挑战大实验（大实验内容来源于你的想法或老师列好的题目，需要与老师协商确定，需完成基本lab，但可不参加闭卷考试），如果有，可直接给老师email或课后面谈。
- 否。因为感觉课程压力较大。

>  

---
