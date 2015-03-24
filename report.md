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
