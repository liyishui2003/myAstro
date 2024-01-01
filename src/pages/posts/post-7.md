---
layout: ../../layouts/MarkdownPostLayout.astro
title: 'MIT6.828 Lab1环境搭建'
pubDate: 2024-01-01
description: 'something about 6.828'
author: 'liyishui'
image:
    url: 'https://pic.imgdb.cn/item/65915597c458853aef276817.jpg'
    alt: '在南京'
tags: ["Os"]
---

## 下载qumu模拟器

我的配置:Ubuntu20、clash

```
git clone https://github.com/geofft/qemu.git -b 6.828-1.7.0
```

使用这条命令要求先下载git和虚拟机能连网(参考资料: https://zhuanlan.zhihu.com/p/380614384 )

报错：fatal: 无法访问 'https://github.com/geofft/qemu.git/'： Failed to connect to 127.0.0.1 port 7890: 拒绝连接，原因是我之前科学上网的遗留问题。到home目录下，点击右上角汉堡菜单(三条杠)打开显示隐藏文件，找到.gitconfig，删掉这个文件即可。

```
cd /home/qemu (或者你的下载路径)
```

下一步输入

```
./configure
```

但是我报错：ERROR: "cc" either does not exist or does not work，这是因为没有下载gcc编译器，输入
```
sudo apt-get install g++
```
下一步报错：ERROR: Python not found. Use --python=/path/to/python，这是因为没有安装python，输入
```
apt-get install python
```
下一步继续报错：ERROR: zlib check failed Make sure to have the zlib libs and headers installed.
这是因为没有下载zlib，输入
```
sudo apt-get install zlib1g-dev
```
下一步继续报错：ERROR: glib-2.12 required to compile QEMU，这是因为缺少glib，输入
```
sudo apt-get install libglib2.0-dev
```
下一步继续报错：ERROR: pixman not present. 这是因为缺少pixman，输入
```
sudo apt-get install libpixman-1-dev
```

下一步继续报错：ERROR: DTC not present. Your options:(1) Preferred: Install the DTC devel package (2) Fetch the DTC submodule, using: git submodule update --init dtc

这个问题有两种解决办法
* 输入
```
./configure --disable-kvm --target-list="i386-softmmu x86_64-softmmu"
```
没看到报错而是一堆配置信息时，说明qemu安装成功
* 见https://www.cnblogs.com/fatsheep9146/p/5068353.html 里的dtc部分，我没有尝试过，是第一个方法成功的

下一步应该输入
```
make && make install
```

但是报错：Command 'make' not found, but can be installed with:，这是因为make工具没有下载，输入
```
sudo apt install make
```

再输入make && make install，看到如下输出，说明编译成功
<img src="https://pic.imgdb.cn/item/6592cfa8c458853aef2dafd5.png" width=700>

标红的地方其实是warning（然而一般都会被我无视，只要先能跑就好。。）在make中的warn都会以warning的方式呈现

## 下载系统代码

(这里是我的心路历程，就不删掉了，但是不要下载这条链接，请往下再翻翻)
输入
```
git clone https://pdos.csail.mit.edu/6.828/2014/jos.git   lab
```
获取MIT提供的操作系统，这段代码会在qemu文件夹下面生成一个lab文件夹，里面有对应代码

```
cd lab(或者进入到你的lab文件夹)
```
输入
```
make
```
报错:
```
***
*** Error: Couldn't find a working QEMU executable.
*** Is the directory containing the qemu binary in your PATH
*** or have you tried setting the QEMU variable in conf/env.mk?
***
***
*** Error: Couldn't find a working QEMU executable.
*** Is the directory containing the qemu binary in your PATH
*** or have you tried setting the QEMU variable in conf/env.mk?
***
+ ld obj/kern/kernel
ld: obj/kern/printfmt.o: in function `printnum':
lib/printfmt.c:41: undefined reference to `__udivdi3'
ld: lib/printfmt.c:49: undefined reference to `__umoddi3'
make: *** [kern/Makefrag:71：obj/kern/kernel] 错误 1
```
参考https://www.coder.work/article/7533729 ，ERROR消失了，但是下面的这几行还没有消失：
```
ld: obj/kern/printfmt.o: in function `printnum':
lib/printfmt.c:41: undefined reference to `__udivdi3'
ld: lib/printfmt.c:49: undefined reference to `__umoddi3'
make: *** [kern/Makefrag:71：obj/kern/kernel] 错误 1
```
原因是开发环境是64而我gcc,但需要的是32位，所以安装32位gcc。输入
```
sudo apt-get install gcc-multilib
```

输入
```
make qemu
```
跳出如下界面，但是屏幕在不断闪烁
<img src="https://pic.imgdb.cn/item/6592d059c458853aef30e4d5.png" width = 700>

说明刚才安装的系统代码不匹配，下载一个新的，输入
```
git clone https://github.com/mit-pdos/xv6-public
```
```
cd xv6-public(或者你的下载路径)
```
然后输入
```
make && make qemu
```
出现如下图像表示你成功在自己的虚拟机里面加载了虚拟机!!终于完成了这个实验的环境配置:D

<img src="https://pic.imgdb.cn/item/6592d0c2c458853aef32be7e.png" width = 700>

(不行了该滚去复习了QAQQBQQCQ)

