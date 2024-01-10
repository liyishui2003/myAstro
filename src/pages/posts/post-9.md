---
layout: ../../layouts/MarkdownPostLayout.astro
title: 'MIT6.828 Lab1-1(分析boot.S文件)'
pubDate: 2024-01-01
description: 'something about 6.828'
author: 'liyishui'
image:
    url: 'https://pic.imgdb.cn/item/65915597c458853aef276817.jpg'
    alt: '在南京'
tags: ["Os"]
---
### 一点小bug
突然发现之前环境配的不全，就按照 https://blog.csdn.net/Alagagaga/article/details/109735168 重新配了一遍环境，被cs144折磨得不清后配这个环境突然感觉非常简单。唯一的坑点是要用gcc32位，否则在make的时候会报错，教程里只给了安装gcc32的命令，但是只安装是不行的。

最后我删除了电脑上所有的gcc，然后单独安装gcc32。
命令如下
```
#打开终端，运行以下命令来卸载 64 位的 GCC：
sudo apt-get remove gcc
#运行以下命令来卸载 64 位的 GCC 库文件：
sudo apt-get autoremove
#如果你需要删除 GCC 的配置文件，可以运行以下命令：
sudo apt-get purge gcc
#下载gcc32
sudo apt-get install gcc-multilib
```

# Exercise 2

进入到/6.828/lab目录下输入下列命令进入调试模式
```
sudo make qemu-gdb
```
然后新开一个终端进入到同样目录，输入
```
sudo make gdb
```
我在这里报错：显示连接localhost26000超时，仔细一看发现qemu运行在25000，gdb要连接的是260000，显然端口号不匹配，查看\lab\GNUmakefile,修改端口号
<img src="https://pic.imgdb.cn/item/659ae129871b83018a78bd1f.jpg">
修改成26000后正常连接，在gdb终端输入si，会得到汇编指令。
以下是输入22次si得到的指令。
```
[f000:fff0]    0xffff0:	ljmp   $0xf000,$0xe05b
[f000:e05b]    0xfe05b:	cmpl   $0x0,%cs:0x6ac8
[f000:e062]    0xfe062:	jne    0xfd2e1
[f000:e066]    0xfe066:	xor    %dx,%dx
[f000:e068]    0xfe068:	mov    %dx,%ss
[f000:e06a]    0xfe06a:	mov    $0x7000,%esp
[f000:e070]    0xfe070:	mov    $0xf34c2,%edx
[f000:e076]    0xfe076:	jmp    0xfd15c
[f000:d15c]    0xfd15c:	mov    %eax,%ecx
[f000:d15f]    0xfd15f:	cli    
[f000:d160]    0xfd160:	cld    
[f000:d161]    0xfd161:	mov    $0x8f,%eax
[f000:d167]    0xfd167:	out    %al,$0x70
[f000:d169]    0xfd169:	in     $0x71,%al
[f000:d16b]    0xfd16b:	in     $0x92,%al
[f000:d16d]    0xfd16d:	or     $0x2,%al
[f000:d16f]    0xfd16f:	out    %al,$0x92
[f000:d171]    0xfd171:	lidtw  %cs:0x6ab8
[f000:d177]    0xfd177:	lgdtw  %cs:0x6a74
[f000:d17d]    0xfd17d:	mov    %cr0,%eax
[f000:d180]    0xfd180:	or     $0x1,%eax
[f000:d184]    0xfd184:	mov    %eax,%cr0
[f000:d187]    0xfd187:	ljmpl  $0x8,$0xfd18f
```
逐个看看
```
[f000:fff0] 0xffff0: ljmp $0xf000,$0xe05b：这条指令是一个长跳转指令，将程序控制权转移到地址 0xf000:e05b 处的指令。这条指令通过加载代码段选择子 $0xf000 和偏移量 $0xe05b 来进行跳转。

[f000:e05b] 0xfe05b: cmpl $0x0,%cs:0x6ac8：这条指令比较值 $0x0 和代码段寄存器 %cs 偏移量为 0x6ac8 的内存位置中的值。

[f000:e062] 0xfe062: jne 0xfd2e1：这条指令表示如果前一条指令的比较结果不相等，则跳转到地址 0xfd2e1 进行执行。

[f000:e066] 0xfe066: xor %dx,%dx：这条指令将寄存器 %dx 的值与自身进行异或操作，将 %dx 的值置为 0。

[f000:e068] 0xfe068: mov %dx,%ss：这条指令将寄存器 %dx 的值复制给栈段寄存器 %ss。

[f000:e06a] 0xfe06a: mov $0x7000,%esp：这条指令将立即数 $0x7000 的值赋给栈指针寄存器 %esp。

[f000:e070] 0xfe070: mov $0xf34c2,%edx：这条指令将立即数 $0xf34c2 的值赋给寄存器 %edx。

[f000:e076] 0xfe076: jmp 0xfd15c：这条指令无条件跳转到地址 0xfd15c 进行执行。

[f000:d15c] 0xfd15c: mov %eax,%ecx：这条指令将寄存器 %eax 的值赋给寄存器 %ecx。

[f000:d15f] 0xfd15f: cli：这条指令禁用中断。

[f000:d160] 0xfd160: cld：这条指令将方向标志位 %df 清零，表明字符串操作将按照递增的方向进行。

[f000:d161] 0xfd161: mov $0x8f,%eax：这条指令将立即数 $0x8f 的值赋给寄存器 %eax。

[f000:d167] 0xfd167: out %al,$0x70：这条指令将寄存器 %al 的值输出到端口 0x70。

[f000:d169] 0xfd169: in $0x71,%al：这条指令从端口 0x71 读取值，并将其赋给寄存器 %al。

[f000:d16b] 0xfd16b: in $0x92,%al：这条指令从端口 0x92 读取值，并将其赋给寄存器 %al。

[f000:d16d] 0xfd16d: or $0x2,%al：这条指令将立即数 $0x2 与寄存器 %al 进行逻辑或操作。

[f000:d16f] 0xfd16f: out %al,$0x92：这条指令将寄存器 %al 的值输出到端口 0x92。

[f000:d171] 0xfd171: lidtw %cs:0x6ab8：这条指令加载中断描述符表的界限，从代码段 %cs 偏移量为 0x6ab8 的内存位置读取。

[f000:d177] 0xfd177: lgdtw %cs:0x6a74：这条指令加载全局描述符表的界限和基址，从代码段 %cs 偏移量为 0x6a74 的内存位置读取。

[f000:d17d] 0xfd17d: mov %cr0,%eax：这条指令将控制寄存器 %cr0 的值赋给寄存器 %eax。

[f000:d180] 0xfd180: or $0x1,%eax：这条指令将立即数 $0x1 与寄存器 %eax 进行逻辑或操作。

[f000:d184] 0xfd184: mov %eax,%cr0：这条指令将寄存器 %eax 的值赋给控制寄存器 %cr0。

[f000:d187] 0xfd187: ljmpl $0x8,$0xfd18f：这条指令是一个长跳转指令，将程序控制权转移到代码段选择子 $0x8 和偏移量 $0xfd18f 处的指令。
```
上面给出的是汇编指令的字面意思，那组合起来到底在做什么呢？

首先设置了ss 和 esp寄存器(对应4、5条)；

然后cli屏蔽了中断

cld则是一个控制字符流向的命令，和后面的in out有关，暂时先不管

再通过in out 和IO设备交互，进行一些初始化，打开A20门

接着lidtw lgdtw两条命令就是加载idtr gdtr寄存器

最后enable %cr0寄存器，进入实模式，长跳转到内核部分执行
这里面涉及到了一些东西，做个笔记：
 * #### PC开始运行时，CS = 0xf000，IP = 0xfff0，对应物理地址为0xffff0.第一条指令做了jmp操作，跳到物理地址为0xfe05b的位置。

 * #### CLI：Clear Interupt，禁止中断发生。STL：Set Interupt，允许中断发生。CLI和STI是用来屏蔽中断和恢复中断用的，如设置栈基址SS和偏移地址SP时，需要CLI，因为如果这两条指令被分开了，那么很有可能SS被修改了，但由于中断，而代码跳去其它地方执行了，SP还没来得及修改，就有可能出错。

 * #### CLD: Clear Director。STD：Set Director。在字行块传送时使用的，它们决定了块传送的方向。CLD使得传送方向从低地址到高地址，而STD则相反。

 * #### 汇编语言中，CPU对外设的操作通过专门的端口读写指令来完成，读端口用IN指令，写端口用OUT指令。

 * #### 地址卷绕：用两个 16 位的寄存器左移相加来得到 20 位的内存地址这里还是有问题。那就是两个 16 位数相加所得的最大结果是超过 20 位的。例如段基址 0xffff 左移变成 0xffff0 和偏移量 0xffff 相加得到 0x10ffef 这个内存地址是“溢出”的，怎么办？这里 CPU 被设计出来一个“卷绕”机制，当内存地址超过 20 位则绕回来。举个例子你拿 0x100001 来寻址，我就拿你当作 0x00001 。你超出终点我就把你绕回起点。

 * #### A20 gate：现代的 x86 计算机，无论你是 32 位的还是 64 位的，在开机的那一刻 CPU 都是以模拟 16 位模式运行的，地址卷绕机制也是有效的，所以无论电脑内存有多大，开机的时候 CPU 的寻址能力只有 1MB，就好像回到 8086 时代一样。 现在计算机都有个“开关”叫 A20 gate，开机的时候 A20 gate 是关闭的，CPU 以 16 位模式运行，当 A20 gate 打开的时候“卷绕”机制失效，内存寻址突破 1MB 限制，我们就可以切换到正常的模式下运行了。

 * #### 从给 x86 通电的一刻开始，CPU 执行的第一段指令是 BIOS 固化在 ROM 上的代码，这个过程是硬件定死的规矩。而 BIOS 在硬件自检完成后（你会听到“滴”的一声）会根据在 BIOS 里设置的启动顺序（硬盘、光驱、USB）读取每个引导设备的第一个扇区 512字节的内容，并判断这段内容的最后 2 字节是否为 0xAA55。如果是说明这个设备是可引导的，于是就将这 512 字节的内容放到内存的 0x7C00 位置，然后告诉 CPU 去执行这个位置的指令。这个过程同样是硬件定死的规矩。
 
 # Exercise 3
在一个终端输入sudo make qemu-gdb开启qemu的调试模式，另一个终端输入make gdb开始调试。
在调试端（为了方便称输入make gdb的端为调试端）输入
```
b *0x7c00
```
显示“Breakpoint 1 at 0x7c00”，表明已经在0x7c00这个位置打了断点。
然后输入
```
x/20i 0x7c00
解释：x/20i 0x7c00 的意思是在内存地址 0x7c00 处开始，打印出接下来 20 条指令的汇编代码。
其中 "x" 表示以十六进制方式输出，"/20i" 表示输出 20 条指令的汇编代码，"0x7c00" 是指起始地址。
这条命令通常被用于在引导扇区代码中调试和查看代码。
```
跳出20条命令,如下图

<img src="https://pic.imgdb.cn/item/659e4227871b83018aae9649.jpg" width=500>

在调试端输入

```
c   //对的，输入单字母c
```

发现qemu端打印出Booting from Hard Disk
<img src="https://pic.imgdb.cn/item/659e433e871b83018ab2f409.jpg" width=500>

这是在做什么？
软盘或者硬盘的存储单元都是512字节的，每512一个字节称为一个扇区。512字节是disk最小的传输粒度,也就是说每次往硬盘写入的或者读出的数据都是512字节的整数倍的。如果一个硬盘是可引导的（bootable），那么他的第一个扇区叫做引导扇区，在这个扇区内部存放着boot loader。
当BIOS发现一个软盘或者一个硬盘是可引导的时候，BIOS会将引导扇区 （boot sector）内的bootloeader复制到内存的0x7c00处（cs:ip=0x0000:0x7c00），然后BIOS会执行一个jmp 0x0000:0x7c00，接着跳转到了引导扇区的代码去执行。

在我们的代码中boot loader 由两个文件组合而成，分别是./boot/boot.S和./boot/main.c,boot loader主要完成下面两个工作：

* Boot loader将处理器从实模式切换到保护模式，因为在实模式之下处理器只能访问0-1MB的内存，切换到保护模式才可以访问更大的内存。
* 接下来boot loader会通过汇编指令从硬盘当中将kernel读入到内存当中。

继续跟踪
```
x/20i 0x7c00
```
得到
<img src="https://pic.imgdb.cn/item/659e4672871b83018ac0290e.jpg">

这些将要执行的代码来自6.828/lab/boot/boot.s文件，现贴出该文件的部分内容和注释。
将从以下几部分分析boot.s代码：初始化段寄存器、打开A20门、从实模式跳到虚模式（需要设置GDT和cr0寄存器。

### 初始化段寄存器 

```
#include <inc/mmu.h>

// .set是AT&T的语法，和x86的equ是一个意思
//就是声明一些常量，在编译的时候会被直接替换为对应的值
.set PROT_MODE_CSEG, 0x8         # kernel code segment selector
.set PROT_MODE_DSEG, 0x10        # kernel data segment selector
.set CR0_PE_ON,      0x1         # protected mode enable flag

//这里的start类似于c语言里的main函数。
//cli意味着从这一刻开始你的计算机将不再响应任何中断事件（比如这时候你敲个键盘点个鼠标啥的，CPU 就不再理你了）。
//之所以要关闭中断响应是因为要保证引导代码的顺利执行（总不能执行到一半被 CPU 给中断了吧，那直接就挂了）。
//.code16 这句。这告诉 CPU 我们目前是在 16 位模式下执行代码，此时内存寻址能力只有 1MB，并且是“实模式”下。
.globl start
start:
  .code16                     # Assemble for 16-bit mode
  cli                         # 清除中断标志
  cld                         # 字符串复制的方向，递增复制

  // Set up the important data segment registers (DS, ES, SS).
  //初始化段寄存器（DS，ES，SS）首先，使用xorw指令将寄存器ax与自身进行异或运算，将结果存储回寄存器ax。
  //这一步的目的是将寄存器ax的值清零，即将其设置为0。接下来，使用movw指令将寄存器ax的值复制到寄存器ds，
  //将数据段寄存器设置为0。同样地，使用movw指令将寄存器ax的值复制到寄存器es和ss，
  //将附加段寄存器和堆栈段寄存器都设置为0。
  //插一句，程序中的数据还要分很多种类型，所以 CPU 针对一个程序准备了 4 个寄存器来存储他们的“段基址”。
  //这 4 个寄存器分别是用于程序指令的 CS 代码段寄存器、用于程序数据的 DS 数据段寄存器、
  //用于程序堆栈（也是数据的一种）的 SS 堆栈段寄存器和 ES 附加段寄存器（也是数据的一种）。
  xorw    %ax,%ax             # Segment number zero
  movw    %ax,%ds             # -> Data Segment
  movw    %ax,%es             # -> Extra Segment
  movw    %ax,%ss             # -> Stack Segment
```
### 打开A20门
```
  //控制 A20 gate 的方法有 3 种：804x 键盘控制器法,Fast A20 法，BIOS 中断法
  //因为是教学设计，就用了最古早的804x键盘控制器法
  // Enable A20:
  //   For backwards compatibility with the earliest PCs, physical
  //  address line 20 is tied low, so that addresses higher than
  //   1MB wrap around to zero by default.  This code undoes this.
//下面是处理键盘的一些工作以及开启A20地址线
//seta20.1和seta20.2两段代码实现打开A20门的功能，其中seta20.1是向键盘控制器的0x64端口发送0xd1命令，这个命令的意思是告诉键盘控制器我要给你的p2写入数据了
//seta20.2是向键盘控制器的 P2 端口写数据了。写数据的方法是把数据通过键盘控制器的 0x60 端口写进去。写入的数据是 0xdf。
//因为 A20 gate 就包含在键盘控制器的 P2 端口中，随着 0xdf 的写入，A20 gate 就被打开了。
seta20.1:
  inb     $0x64,%al               # Wait for not busy
  testb   $0x2,%al
  jnz     seta20.1

  movb    $0xd1,%al               # 0xd1 -> port 0x64
  outb    %al,$0x64

seta20.2:
  inb     $0x64,%al               # Wait for not busy
  testb   $0x2,%al
  jnz     seta20.2

  movb    $0xdf,%al               # 0xdf -> port 0x60
  outb    %al,$0x60
```

### 从实模式切换到保护模式
```
  // 加载GDT到gdtr寄存器，并且将cr0寄存器的最低位设置为1
  // 这样就开启了保护模式
  lgdt    gdtdesc
  movl    %cr0, %eax
  orl     $CR0_PE_ON, %eax
  movl    %eax, %cr0
  
  //跳转到保护模式下的代码去执行
  ljmp    $PROT_MODE_CSEG, $protcseg
   .code32                     # Assemble for 32-bit mode
protcseg:
  // Set up the protected-mode data segment registers
  movw    $PROT_MODE_DSEG, %ax    # Our data segment selector
  movw    %ax, %ds                # -> DS: Data Segment
  movw    %ax, %es                # -> ES: Extra Segment
  movw    %ax, %fs                # -> FS
  movw    %ax, %gs                # -> GS
  movw    %ax, %ss                # -> SS: Stack Segment
```
这段代码知识密度极大，所以我把注释单独拎出来放在下面写了，并且会拆分成2个部分进行分析。
另，以下资料主要来自于： https://leenjewel.github.io/blog/2014/07/29/%5B%28xue-xi-xv6%29%5D-cong-shi-mo-shi-dao-bao-hu-mo-shi/  ，加入一点个人理解。

##### 第一部分：加载GDT到gdtr寄存器
```
lgdt    gdtdesc
```
首先什么是GDT?
在保护模式下，内存管理有两种方式：分页式和分段式。我们先只讨论分段式。

在分段模式下，内存被划分为很多个“片段”，程序数据以及指令就放在这些片段中，当要读取内存中具体的数据时，首先要知道这个数据在哪个“片段”里，这时段寄存器里的“段基址”指向某一个内存片段的下标，而这时的“偏移量”则相应的表示为具体的数据在它所在的内存“片段”里的偏移量。

好比你家在第三小区走进去第五栋楼，通过段基址+偏移量我们就能定位到数据的具体位置。

所以在分段模式下，内存里会有一个“表”，这个“表”里存放了每个内存“片段”的信息（如这个“片段”在内存中的地址，这个“片段”多长等），比如我们现在将内存分成 10 个片段，则这时我们有一个“表”，这个“表”有 10 项，分别存放着对这 10 个内存片段的描述信息。

这时我有个数据存放在第 5 个片段中，在第 5 个片段的第 6 个位置上，所以当我们想要读取这个数据的时候，我们的数据段寄存器里存放的“段基址”是 5 这个数，代表要去第 5 个片段上找，那第五个片段在哪里？就要问这个表，表存储着片段的位置和长度。找到了第五段以后再去找第6个位置，就能得到数据。

而要想实现在分段式保护模式下成功的寻址，操作系统需要做的就是在内存中建立这个“表”，“表”里放好内存分段的描述信息，然后把这个“表”在内存的什么位置，以及这个“表”里有多少个内存分段的描述信息告诉 CPU。

这个“表”就是GDT。

有了这些知识以后我们可以用更规范的描述来说明GDT：GDT是全局描述符表，GDTR是全局描述符表寄存器。想要在“保护模式”下对内存进行寻址就先要有 GDT，GDT表里每一项叫做“段描述符”，用来记录每个内存分段的一些属性信息（这些信息就是刚才提到的片段所在的地址和长度等），每个段描述符占8字节。

但是其实GDT本身也是一些数据，GDT要存放在哪里???

CPU使用GDTR寄存器来保存我们GDT在内存中的位置和GDT的长度。lgdt gdtdesc将源操作数的值（存储在gdtdesc地址中）加载到全局描述符表寄存器中。

GDTR 寄存器一共 48 位，其中高 32 位用来存储我们的 GDT 在内存中的位置，其余的低 16 位用来存我们的 GDT 有多少个段描述符。 16 位最大可以表示 65536 个数，这里我们把单位换成字节，而一个段描述符是 8 字节，所以 GDT 最多可以有 8192 个段描述符。不仅 CPU 用了一个单独的寄存器 GDTR 来存储我们的 GDT，而且还专门提供了一个指令用来让我们把 GDT 的地址和长度传给 GDTR 寄存器，这就是上面代码里的：
```
lgdt    gdtdesc
```
gdtdesc的定义在boot.s代码底部，.long是GDT真实的物理地址，.word是GDT的长度
<img src="https://pic.imgdb.cn/item/659e846d871b83018ab1b309.jpg">


##### 第二部分：通过控制cr0切换导保护模式
```
movl    %cr0, %eax
orl     $CR0_PE, %eax
movl    %eax, %cr0
```
要进入“保护模式”我们也需要打开一个开关，这个开关叫“控制寄存器”，x86 的控制寄存器一共有 4 个分别是 CR0、CR1、CR2、CR3，而控制进入“保护模式”的开关在 CR0 上，这四个寄存器都是 32 位的，我们看一下 CR0 上和保护模式有关的位
<img src="https://pic.imgdb.cn/item/659e87c3871b83018ac33542.jpg" width=700>
* PG:为 0 时代表只使用分段式，不使用分页式；为 1 是启用分页式
* PE:为 0 时代表关闭保护模式，为 1 则开启保护模式

那么这三行代码的意思就是：
* 用通用寄存器 eax 来保存 cr0 寄存器的
* 然后 CR0_PE 这个宏的定义在 mmu.h 文件中，是个数值 0x00000001，将这个数值与 eax 中的 cr0 寄存器的值做“或”运算后，就保证将 cr0 的第 0 位设置成了 1 即 PE = 1 保证打开了保护模式的开关
* 将新的计算后的 eax 寄存器中的值写回到 cr0 寄存器中就完成了到保护模式的切换

### 32位的时代！！预备，跳
```
ljmp    $PROT_MODE_CSEG, $protcseg
.code32                     # Assemble for 32-bit mode
protcseg:
  // Set up the protected-mode data segment registers
  movw    $PROT_MODE_DSEG, %ax    # Our data segment selector
  movw    %ax, %ds                # -> DS: Data Segment
  movw    %ax, %es                # -> ES: Extra Segment
  movw    %ax, %fs                # -> FS
  movw    %ax, %gs                # -> GS
  movw    %ax, %ss                # -> SS: Stack Segment
```
.set PROT_MODE_CSEG, 0x8 这句已经定义了PROT_MODE_CSEG是一个常量0x8，$protcseg则在下面给出了定义。也就是说，protcseg是一个标签（label），用于标识一个代码段的起始位置。它并不是一个跳转目标，而是一个可以在代码中被引用的标记。

这些mov语句先把PROT_MODE_DSEG的值搬到ax上，再把ax上的该值搬运到DS/ES/FS/GS/SS上。
PROT_MODE_DSEG的定义在boot.s的开头定义过了，是一个常量0x10。
```
.set PROT_MODE_DSEG, 0x10 
```
这里没找到很详细的资料，这些movw语句应该是在做一些初始化的部分
### 调用bootmain函数，进入c语言的时代
```
  // Set up the stack pointer and call into C.
  movl    $start, %esp
  call bootmain
```
到这里，就开始加载bootmain函数了。
