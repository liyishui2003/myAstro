---
layout: ../../layouts/MarkdownPostLayout.astro
title: '记一次失败的CS144 Lab0尝试'
pubDate: 2024-01-03
description: 'something about 6.828'
author: 'liyishui'
image:
    url: 'https://pic.imgdb.cn/item/65915597c458853aef276817.jpg'
    alt: '在南京'
tags: ["Os"]
---
#### 下载前置软件包
要下载的软件包一共有：

<img src="https://pic.imgdb.cn/item/659ab503871b83018ab08c91.png"> 

具体怎么下载，我是先用whereis xxx（你的软件名）看下自己有没有安装该软件，没有的话一般用sudo apt-get install xxx就可以了。单子里列出来的包很多在装虚拟机时自带了或者会安，所以先whereis一下免去不少麻烦。

其中有的就跳过。以下是安装其他软件包的历程，先安了clang-6.0、clang-format-6.0、libpcap-dev

```
sudo apt-get install clang-6.0
sudo apt-get install clang-format-6.0
sudo apt-get install libpcap-dev
```
然后安装cmake我参考的是这篇文章 https://blog.csdn.net/qq_38327353/article/details/107528837 ，用的方法三

安装mininet:
```
git clone https://gitee.com/derekwin/mininet.git
./mininet/util/install.sh -a
```

报了和python有关的错，在这里py报错的话基本是版本问题，如果是py2就不行。进入mininet/example目录，输入python查看python版本，发现是2.7.18，显然不对。解决方法是用下面四条命令建立软连接，前两条命令是删除，后两条是让python和python3都指向/usr/bin/python3.8（这里要换成自己python3的位置）。输入python命令检查，切换成功。

```
liyishui@M102102103:~/mininet/examples$ sudo rm -rf /usr/bin/python3
liyishui@M102102103:~/mininet/examples$ sudo rm -rf /usr/bin/python
liyishui@M102102103:~/mininet/examples$ sudo ln -s /usr/bin/python3.8 /usr/bin/python
liyishui@M102102103:~/mininet/examples$ sudo ln -s /usr/bin/python3.8 /usr/bin/python3
liyishui@M102102103:~/mininet/examples$ python
Python 3.8.10 (default, May 26 2023, 14:05:08) 
[GCC 9.4.0] on linux
```
但是还是没有解决问题，我定睛一看，发现报的错是：
/usr/bin/python: No module named pip，这是没有安装pip库的意思，安一下python3-pip。
再试一遍，出现"Enjoy Mininet!"的字样说明安装mininet成功。

#### 下载实验资料

```
git clone --recursive https://github.com/Kiprey/sponge.git
mkdir -p sponge/build
cd sponge/build
cmake ..
make
```
这里我原本直接用git(git clone --recursive git@github.com:Kiprey/sponge.g)，访问被拒绝了，换成http方式就可行。

#### Lab0开始
```
telnet cs144.keithw.org http
```
输入上述代码，连接远程网页服务器，成功后出现下图界面：

接下来应该 *快速* 地输入以下代码
```
GET /hello HTTP/1.1
Host: cs144.keithw.org
Connection: close

```
遇到报错：

<img src="https://pic.imgdb.cn/item/659ab503871b83018ab08ae7.png">

冥思苦想终于想到也许不只是这个网站上不去，试了一下google，果然也上不去！但是之前下载国外软件包时又很快速，所以给我一种我的虚拟机能翻墙的错觉。其实是：之前换过镜像源->下载国外东西时用的是国内镜像，所以很快->误以为能翻墙->但是一直"Connection closed by foreign host."，所以接下来给虚拟机配置共享VPN。
最终用了桥接模式： https://blog.csdn.net/qq_27462573/article/details/130484723 ，成功了，虚拟机可以访问外网。在linux里用firefox浏览器访问http://cs144.keithw.org/hello也是正常的。
但是仍然报刚才的错。
telnet的对象换成连接百度，连接成功：200

<img src="https://pic.imgdb.cn/item/659ab502871b83018ab08819.png">


### 和自己通信
输入
```
telnet localhost 9090
```
惨遭拒绝
```
Trying 127.0.0.1...
telnet: Unable to connect to remote host: Connection refused
```
尝试直接
```
telnet localhost
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
Connection closed by foreign host.
```
发现原来没有安装telnet服务，上面可以使用telnet命令误以为自己安装了
配置完毕之后启动xtind服务，仍然报错，查看防火墙规则，发现没有开放23端口
输入查看防火墙情况
```
sudo iptables -L -n
```

开放23端口
```
sudo iptables -A INPUT -p tcp --dport 23 -j ACCEPT
```

但是还是失败。到这里的问题非常明确了，不只是连接cs144的网站被拒接，连接localhost也被拒绝，并且确认我的xinetd服务打开、端口有在监听、防火墙开放，理论上应该没有bug了。
最后还是检查了一遍自己安xinetd时的各种配置，重新装，突然就可以和localhost联通了，可能是深夜配环境时脑子不清楚，有细节没做好。
telnet localhost成功（话说自己登录自己的感觉好神奇），但是telnet cs144时仍然失败，先放着。

输入
```
nc -v -l -p 9090
```
报错：无法解析，是因为服务商没有提供dns解析服务，临时解决办法是修改etc/resolv.conf，加入nameserver 8.8.8.8 ，但是重启后就会失败，因为这个文件是动态生成的，永久修改我尝试了百度的办法，因为失败暂且不表。
等到出现Listening on 0.0.0.0 9090时，新开一个终端，输入
```
telnet localhost 9090
```
此时前一个终端显示：Connection received on localhost 47900，一台机子的两个终端就可以开始通信了。
随便输入一点东西就可以在另一个终端看到。
在终端输入ctrl+c关闭连接。

### 向qq邮箱发邮件
输入
```
telnet smtp.qq.com smtp
//跳出一些信息，下面省略不表
HELO qq.com
//
AUTH LOGIN
//
这里输入邮箱的base64编码，也就是xxx@qq.com转化成base64
这里输入邮箱的授权码
跳出AUTH successful后可以按照下图的格式填写邮件内容
```

<img src="https://pic.imgdb.cn/item/659ab54e871b83018ab1f6e1.png">

### code

编程部分我是安装了vscode，参考： https://www.baidu.com/link?url=vdIdAsoCMspEOAv47WhuRt1AbltA83OOPwptXWrUcl8jAsy6acN3n4d1gL3Y-xNRBbfLozEGCeKiBrgfthGchVr9OHXZFmPSrvsHK7qRg1S&wd=&eqid=8891bee6007d730d00000005659956ae

cd进入sponge文件夹，输入code .就会跳出vscode界面，编辑weight.cc代码
补全geturl函数
```
void get_URL(const string &host, const string &path) {
    Address address(host, "http"); //传入主机的host 和需要的服务
    TCPSocket socket; //创建一个socket对象
    socket.connect(address);    // socket对象和服务器连接
    // 利用字符串拼接，参考2.1.2构建一个http请求报文
    socket.write("GET " + path + " HTTP/1.1\r\n"); // \r\n 表示回车
    socket.write("HOST: " + host + "\r\n");
    socket.write("\r\n");
    socket.shutdown(SHUT_WR);     // request结束
    // content
    while (!socket.eof()) {      //如果管道没有关，持续读输入进来的数据
        std::cout << socket.read(1);
    }
    socket.close();     // 关闭socket
}
```
还是报错，看来刚才连不上cs144在这里产生了影响，之前没有解决最大的问题在于没有报错，反馈很少，那么我们就主动去看日志。发现，嗯，在var/log/下面没有message，本来应该来这里看的，根据 https://blog.csdn.net/feinifi/article/details/107037892 配置一下，看到有message日志输出了。
但是很遗憾，我的日志输出并没有给我什么帮助，最后还做了以下尝试
* #### 在linux里用wireshark抓包，但是没分析出什么
* #### 新建了一个虚拟机，配置cs144所需的环境，在telnet这步仍然被直接拒绝
* #### 新建了一个wls2，运行脚本配置文件，仍然在telnet这步被直接拒绝

<img src="https://pic.imgdb.cn/item/659ab8e1871b83018ac3b41c.jpg" width = 700>
