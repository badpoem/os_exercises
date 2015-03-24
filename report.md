练习1

1.操作系统镜像文件ucore.img是如何一步一步生成的？(需要比较详细地解释Makefile中每一条相关命令和命令参数的含义，以及说明命令导致的结果)

- 采用gcc –o –c 编译现有的.c文件生成一系列.o文件
- 采用ld指令将这些目标文件转化成一个执行程序bootblock.out
- 采用dd指令生成一个虚拟空间，输出到ucore.img

- 具体如下：
- 生成ucore.img的指令如下：
```
$(UCOREIMG): $(kernel) $(bootblock)
  $(V)dd if=/dev/zero of=$@ count=10000
  $(V)dd if=$(bootblock) of=$@ conv=notrunc
  $(V)dd if=$(kernel) of=$@ seek=1 conv=notrunc
```
- 首先是生成kernel 和 bootblock
```
$(bootblock): $(call toobj,$(bootfiles)) | $(call totarget,sign)
    @echo + ld $@
    $(V)$(LD) $(LDFLAGS) -N -e start -Ttext 0x7C00 $^ -o $(call toobj,bootblock)
    @$(OBJDUMP) -S $(call objfile,bootblock) > $(call asmfile,bootblock)
    @$(OBJCOPY) -S -O binary $(call objfile,bootblock) $(call outfile,bootblock)
    @$(call totarget,sign) $(call outfile,bootblock) $(bootblock)
```
- 生成bootblock 首先生成bootasm.o bootmain.o sign
- bootasm.o由bootasm.S生成
- gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootasm.S -o obj/boot/bootasm.o
- bootmain.o由bootmain.c生成
- gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootmain.c -o obj/boot/bootmain.o
- 生成sign
- gcc -Itools/ -g -Wall -O2 -c tools/sign.c -o obj/sign/tools/sign.o
- gcc -g -Wall -O2 obj/sign/tools/sign.o -o bin/sign
- 参数解释如下：
- -ggdb 生成可供gdb使用的调试信息
- -m32 生成适用于32位环境的代码
- -gstabs 生成stabs格式的调试信息
- -nostdinc 不使用标准库
- -fno-stack-protector 不生成用于检测缓冲区溢出的代码
- -Os 为减小代码大小而进行优化
- -I<dir> 添加搜索头文件的路径
- -fno-builtin 除非使用_builtin_前缀，否则不进行builtin函数的优化
- 生成bootblock.o
- ld -m elf_i386 -nostdlib -N -e start -Ttext 0x7C00 obj/boot/bootasm.o obj/boot/bootmain.o -o obj/bootblock.o
- -m <emulation> 模拟为i386上的连接器
- -nostdlib 不使用标准库
- -N 设置代码段和数据段均可读写
- -e <entry> 指定入口
- -Ttext 指定代码段开始位置
- 然后拷贝二进制代码bootblock.o到bootblock.out
- -S 移除所有符号和重定位信息
- -O <bfdname> 指定输出格式
- 然后使用sign工具处理bootblock.out得到bootblock
```
$(kernel): tools/kernel.ld
    $(kernel): $(KOBJS)
    @echo + ld $@
    $(V)$(LD) $(LDFLAGS) -T tools/kernel.ld -o $@ $(KOBJS)
    @$(OBJDUMP) -S $@ > $(call asmfile,kernel)
    @$(OBJDUMP) -t $@ | $(SED) '1,/SYMBOL TABLE/d; s/ .* / /; /^$$/d' > $(call symfile,kernel)
```
- 生成kernel
- ld -m elf_i386 -nostdlib -T tools/kernel.ld -o bin/kernel obj/kern/init/init.o obj/kern/libs/readline.o obj/kern/libs/stdio.o obj/kern/debug/kdebug.o obj/kern/debug/kmonitor.o obj/kern/debug/panic.o obj/kern/driver/clock.o obj/kern/driver/console.o obj/kern/driver/intr.o obj/kern/driver/picirq.o obj/kern/trap/trap.o obj/kern/trap/trapentry.o obj/kern/trap/vectors.o obj/kern/mm/pmm.o obj/libs/printfmt.o obj/libs/string.o
- -T <scriptfile> 让连接器使用指定的脚本
- 得到bootblock和kernel后
- dd if=/dev/zero of=bin/ucore.img count=10000
- dd if=bin/bootblock of=bin/ucore.img conv=notrunc
- dd if=bin/kernel of=bin/ucore.img seek=1 conv=notrunc\
- 生成一个有10000个块的文件，每个块默认512Bytes，并用0填充
- 把bootblock的内容写到第一个块
- 从第二个块开始写kernel中的内容

>

2.一个被系统认为是符合规范的硬盘主引导扇区的特征是什么？

- 由sign.c可以知道，一个硬盘主引导扇区大小为512Bytes, 且最后两个字节为0x55AA.

>

练习2

1.从CPU加电后执行的第一条指令开始，单步跟踪BIOS的执行。

- 设置单步跟踪后执行，完成单步跟踪，步骤为修改lab1/tools/gdbinit为
```
file bin/kernel
set architecture i8086
target remote :1234
break kern_init
continue
```
- 然后执行make debug，进入gdb调试界面后输入si即可

>

2.在初始化位置0x7c00设置实地址断点,测试断点正常。

- 修改gdbinit为
```
file bin/kernel
set architecture i8086
target remote :1234
break *0x7c00
continue
x /2i $pc
```
- make debug后得到
- 0x7c00:	cli
- 0x7c01:	cld

- 0x7c02:	xor	%ax,%ax
- 0x7c04:	mov	%ax,%ds
- 断点测试正常。

>

3.从0x7c00开始跟踪代码运行,将单步跟踪反汇编得到的代码与bootasm.S和 bootblock.asm进行比较。
- 沿用第2题的修改，逐步使用si命令跟踪代码运行，并使用x /2i $pc指令得到反汇编结果。
- 进行比较后结论为反汇编得到的代码与bootasm.S和bootblock.asm中的代码相同。

>

4.自己找一个bootloader或内核中的代码位置，设置断点并进行测试。
- 已设置并进行测试

>

练习3

BIOS将通过读取硬盘主引导扇区到内存，并转跳到对应内存中的位置执行bootloader。请分析bootloader是如何完成从实模式进入保护模式的。

```
cli
cld
xorw %ax, %ax
movw %ax, %ds
movw %ax, %es
movw %ax, %ss
```
- 清理环境，置零flag和段寄存器 
```
testb $0x2, %al
jnz seta20.1

movb $0xd1, %al
outb %al, $0x64

seta20.1:
inb $0x64, %al 
testb $0x2, %al
jnz seta20.1

movb $0xdf, %al
outb %al, $0x60
```
- 开启A20
```
lgdt gdtdesc
```
- 初始化GDT表
```
movl %cr0, %eax
orl $CR0_PE_ON, %eax
movl %eax, %cr0
```
- 进入保护模式
```
ljmp $PROT_MODE_CSEG, $protcseg
.code32
protcseg:
```
- 通过长跳转更新cs的基地址
```
movw $PROT_MODE_DSEG, %ax
movw %ax, %ds
movw %ax, %es
movw %ax, %fs
movw %ax, %gs
movw %ax, %ss
movl $0x0, %ebp
movl $start, %esp
```
- 设置段寄存器并建立堆栈
```
call bootmain
```
- 进入boot主方法

>

练习4

通过阅读bootmain.c，了解bootloader如何加载ELF文件。通过分析源代码和通过qemu来运行并调试bootloader&OS，
(1)bootloader如何读取硬盘扇区的？
(2)bootloader是如何加载ELF格式的OS？

- (1)static void readsect(void *dst, uint32_t secno);
- readsect函数实现硬盘扇区的读取，其内部使用内联汇编实现outb，采用IO寻址方式将外设的数据读入到内存
- static void readseg(uintptr_t va, uint32_t count, uint32_t offset);
- 而readseg函数对readsect进行了包装，以在bootmain函数中进行调用。

- (2)void bootmain(void);
- bootmain函数实现了ELF格式的判断和读取：
- 首先通过readseg函数读取ELF的头部
- 然后通过ELFHDR的e_magic成员判断该文件是否为ELF格式
- 当其为ELF格式时，从其头部的描述ELF文件应存到内存中什么位置的描述表的头地址（ELFHDR+ELFHDR->e_phoff）开始,使用readseg函数依次将其中内容加载到内存，最后根据ELFHDR->e_entry找到内核的入口

>

练习5

我们需要在lab1中完成kdebug.c中函数print_stackframe的实现，可以通过函数print_stackframe来跟踪函数调用堆栈中记录的返回地址。

- 在kdebug.c文件中依照函数print_stackframe的注释要求完成代码。
- 其中需要注意对ebp进行判断，ebp为0时候堆栈已超过0x7c00，需要break跳出循环。

- 最后一行输出为：
```
ebp is 0x00007bf8, eip is 0x00007d68, argu[0] is 0c031fcfa, argu[1] is 0c08ed88e, argu[2] is 064e4d08e, argu[3] is 0fa7502a8, 
   <unknown>: -- 0x00007d67 --
```
- 对应栈底，即第一个使用堆栈的函数bootmain，由于bootloader设置堆栈从0x7c00开始，而使用call bootmain调用bootmain，故call指令压栈， ebp:0x7bf8对应为bootmain

>

练习6

1.中断描述符表（也可简称为保护模式下的中断向量表）中一个表项占多少字节？其中哪几位代表中断处理代码的入口？

- 由gatedesc定义可知一个表项占用8字节，其中0-1和6-7字节为段偏移：unsigned gd_off_15_0 : 16; unsigned gd_off_31_16 : 16; 2-3字节为段选择子：unsigned gd_ss : 16; 4-5字节定义中断类型等内容。段选择子+段偏移得到中断处理的入口地址。

>

2.请编程完善kern/trap/trap.c中对中断向量表进行初始化的函数idt_init。在idt_init函数中，依次对所有中断入口进行初始化。使用mmu.h中的SETGATE宏，填充idt数组内容。每个中断的入口由tools/vectors.c生成，使用trap.c中声明的vectors数组即可。

- 见代码

>

3.请编程完善trap.c中的中断处理函数trap，在对时钟中断进行处理的部分填写trap函数中处理时钟中断的部分，使操作系统每遇到100次时钟中断后，调用print_ticks子程序，向屏幕上打印一行文字”100 ticks”。

- 见代码

>
