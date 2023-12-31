---
layout: ../../layouts/MarkdownPostLayout.astro
title: '30dayOs_day01'
pubDate: 2023-12-04
description: 'something about myOs'
author: 'liyishui'
image:
    url: 'https://pic.imgdb.cn/item/65915597c458853aef276817.jpg'
    alt: '在南京'
tags: ["Os"]
---
今天拆箱后看着软驱和软盘感觉好激动，老古董欸~

把软盘放进软驱，软驱连接电脑，按照教程格式化硬盘，注意不要勾选快速格式化。

在运行!cons_9x.bat输入install时出了问题，作者的设计是在这里应该把映像文件写进软盘，然而tolset\z_tools\imgtol.com是16位程序，和64位版本的window不兼容，无法运行。

我默默看了自己刚买的两张软盘和一块软驱。软盘一时半会应该是没用了，还有软驱也是。不过可以留一张当纪念，另一张留着看有没有什么其他玩法。

接下来的思路本是用vm新建虚拟机，把映像文件img放进去，作者却在文章的下一页提到：
如果不想买软驱和软盘的话，我还准备了一个模拟器~buling~唉我真是个笨蛋

运行模拟器，终于hello world了

<img src="https://pic.imgdb.cn/item/65915638c458853aef29df45.jpg">

接下来的篇幅作者简单讲述了什么是 **cpu** 一块集成电路板，接收01信号，给出对应指令，输出相应结果

刚才是用二进制编辑器造出了img文件，作者又用汇编程序造了一个功能一样的img文件

<img src="https://pic.imgdb.cn/item/65915791c458853aef2f21a7.jpg" width=700>

编译这段汇编代码，得到一个一模一样的hellos.img文件，再输入run，得到和刚才一样“hello world”的结果

逐行看看这段汇编做了什么

#### DB 0xeb, 0x4e, 0x90, 0x48, 0x45, 0x4c, 0x4c, 0x4f

DB指令是“define byte"的缩写，表示写入一个字节的指令。一个字节的指令刚好就是后面的这串"0xeb, 0x4e, 0x90, 0x48, 0x45, 0x4c, 0x4c, 0x4f"。0xEB JMP 是一条跳转指令，后面的二个字节是偏移量。也就是说0xEB, 0x4E: 这可能是一条跳转指令，0xEB 表示跳转指令的操作码，而 0x4E 可能是跳转的相对偏移量。而0x90, 0x48, 0x45, 0x4C, 0x4C, 0x4F这些十六进制值对应于 ASCII 字符，表示字符串 "HELLO"。

#### DB	0x49, 0x50, 0x4c, 0x00, 0x02, 0x01, 0x01, 0x00

0x49: 对应 ASCII 字符 "I" 0x50: 对应 ASCII 字符 "P" 0x4C: 对应 ASCII 字符 "L"

和上面一行综合起来，说的是这个引导区的名字叫做"HELLOIPL"，后面跟着的是引导区的配置（比如每个扇区大小、根目录的大小等等），学识尚浅，再往下就看不懂了。这里作者想让我们掌握的知识是DB指令和RESB指令。

RESB:是“reserve byte”的略写，如果想要从现在的地址开始空出10个字节来，就可以写成RESB
10，意思是我们预约了这10个字节,而且不仅仅是把指定的地址空出来，它还会在空出来的地址上自动填入0x00。

作者又用优化后的汇编代码写了一份一样的映射img文件
<img src="https://pic.imgdb.cn/item/65916146c458853aef5778d4.jpg" width = 700 height =  300>
<img src="https://pic.imgdb.cn/item/659161d2c458853aef59c4fe.jpg" width = 700 height =  700>
<img src="https://pic.imgdb.cn/item/65916212c458853aef5ae056.jpg" width = 700 height = 500>

这里有几个新知识
* 汇编里";"表示注释，类似c语言中的//

* DW指令表示“define word"，DD指令表示"ddefine double-world"，它们都和DB是姐妹。"word"指16位，也就是2个字节，"double-word"是32位，也就是4个字节。

* RESB 0x1fe-& ，这里的RESB上面说过了，是自动填充。$是目前的字节数，是一个动态的值。0x1fe-&的意思就是用0x1fe这个值减去$(现在的字节数)，然后得到一个结果，填充这么多位的0x00。也就是说，可以动态地填充0，填到0x001fe为止。

* 0x55和0xAA 为什么填充完末尾还要写入0x55和0xAA？计算机启动的时候，会从最初一个扇区读取设备，然后检查这个扇区的最后两个字节，即第511和512字节。如果最后两个字节是0x55和0xAA，那它就认为这个扇区的开头是启动程序，并开始执行这个程序，否则就会报一个不能启动的错误。至于为什么是0x55和0xAA，还得问问设计者。也就是说，到这里，我们已经来到了启动区的结尾。

* 启动区 软盘的第一个扇区称为启动区，计算机读💾时是以512个字节为单位读写的，512字节就称为一个扇区。读软盘是从第一个扇区开始的。确认了启动区的末尾有0x55和0xAA以后，就开始执行该程序。

day1的内容到这里结束，另，今天是2023.12.31，祝大家新年快乐~

<img src="https://pic.imgdb.cn/item/65916ccac458853aef8dc4e5.jpg" width = 700>

