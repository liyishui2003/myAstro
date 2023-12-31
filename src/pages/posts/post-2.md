---
layout: ../../layouts/MarkdownPostLayout.astro
title: Python语言实现两台计算机用TCP协议跨局域网通信
author: liyishui
description: "简单的用python在两台主机中建立通信，想做CS144"
image:
    url: "https://img.zcool.cn/community/010d115bbb689ca8012099c84119c6.jpg@3000w_1l_0o_100sh.jpg"
    alt: "Thumbnail of Astro arcs."
pubDate: 2023-12-30
tags: ["Linux"]
---

## 成果展示

<Image src="https://pic.imgdb.cn/item/658f9fb1c458853aef42506d.png"  width="900" height="500"/>

## 设备要求和实现的功能：
实现的功能：
跨局域网通信（仅支持两台计算机）
跨局域网收发小文件,支持缓存在服务器，再一键接收（仅支持两台计算机）

## 使用方法：
在服务器上运行server.py程序，在两台客户机上分别运行client.py程序，就会弹出图形化界面，就可以开始愉快使用啦。需要修改的地方是client.py里HOST要设置为服务器的ip地址，以及两处toFile要换成对应的服务器路径和本地存储路径。服务器的代码使用了python的tk库，但我用的是华为云的ECS服务器，是linux内核，并不支持跳出图形化界面，要跳出图形化界面的话需要下载X11,把消息转发到本地，在本地才能弹出窗口。具体的配置链接请看[https://www.cnblogs.com/yyiiing/p/17912650.html#5240133]

## 通信的基本原理：
因为临近考试了，虽然感觉很好玩，但没有花太多时间折腾，就搓了两天hh。原理很简单，用服务器作为中转，两台客户机和服务器建立起TCP连接，客户机A发消息给服务器，服务器再转发给客户机B。发文件的原理其实也是一样的，只不过是客户机A先把文件传输到服务器上，然后客户机再从服务器上读取。

## UI界面介绍：
右边蓝色框仅显示聊天的文本信息，左边黄色框显示文件传输信息。
send Text按钮是用来发送文本的
select File按钮是用来选择文件的，点击后会出现如下界面：
send File按钮是用来发送文件的
receive按钮是用来接收所有未接收的文件

## 程序实现的基本原理
### 客户端：
通信部分由于py库已经封装得很好了，所以直接调用socket库去建立和服务器的连接即可。
```
   HOST = '你的服务器IP地址'
    PORT = 21567
    ADDR = (HOST, PORT)
    BUFSIZ = 65536

    tcpCli = socket(AF_INET, SOCK_STREAM)
    tcpCli.connect(ADDR)

    root = tk.Tk()
    chat_window = ChatWindow(root)
    threading.Thread(target=recv, args=(tcpCli, BUFSIZ, chat_window)).start()  # 创建接收消息的线程并启动
    root.mainloop()

    tcpCli.close()
```
比较困难的地方在于收发消息（广义的消息，包括文件和文本）
我建了一个message_queue用来存放所有要放出去的消息
* 出队时检查一下标识符，如果是以**sendFile**开头的话，那它的构成就是:**#sendFile#filepath**(准备要发送的文件在本地的路径)。根据路径读取文件，加上#fileData#告诉服务器我这是要发送文件了，打包发过去即可。
* 如果是文本消息的话不需要加标识符，直接发送。

是的，标识符是我为了区别文件和文本自己设计的，我猜测真实的场景里发送文件和发送消息应该是直接分开的，而不是都塞在一个消息队列里。
```
def send_messages(self):
        while True:
            message = self.message_queue.get()  # 从队列中获取消息
            if message.startswith("#sendFile#"):
                _, __, filepath = message.split("#")
                path_parts = filepath.split("/")
                # 获取最后一个部分
                last_part = path_parts[-1]
                toFile = "/home/liyishui/" #发送到服务器上的存储路径
                toFile+=last_part
                with open(filepath, "rb") as f:
                    data = f.read()
                    tcpCli.sendall(("#fileData#%s#" % toFile).encode() + data)
                toFile= "#sendFile#"+last_part
                tcpCli.send(toFile.encode())
            else:
                tcpCli.send(message.encode())
```

对于接收的消息：
* 如果是以 **#fileData#** 开头,表明这是服务器在向你转发文件，你的客户端将会读取文件名(filepath）和数据(filedata)，进行写入操作，并在左侧消息栏显示 **[Download] File download complete**
* 如果是以 **#sendFile#** 开头，表明这是对方在向你的客户机发送文件，发送完成之后对方朝服务器发送了 **#sendFile#filename**，表示发送成功。服务器再转告你，你的左侧消息栏就会弹出一条 **[Newfile] fileName**
* 否则就是常规的文本消息操作，直接打印到图形化界面即可
```
def recv(sock, BUFSIZ, chat_window):
    try:
        while True:
            data = sock.recv(BUFSIZ)
            print(data)
            print(data.decode())
            if not data:
                break
            if data.startswith(b"#fileData#"):
                _, filename, filepath, filedata = data.decode().split("#", 3)
                toFile = "C:\\Users\\33245\\Desktop\\ttssmm\\animal\\backend\\user2\\" + filepath.strip() #本地文件存储路径
                with open(toFile, "wb") as f:
                    f.write(filedata.encode())
                time_tag = '[%s] ' % ctime()
                tag_config = ('sent_time_tag', 'green')
                chat_window.info_text.config(state='normal')
                chat_window.info_text.insert(tk.END, time_tag, tag_config)
                chat_window.info_text.insert(tk.END, '[Download] File download complete: {}\n'.format(filepath))
                chat_window.info_text.config(state='disabled')
                chat_window.info_text.see(tk.END)
                continue
            elif data.startswith(b"#sendFile#"):
                _, __, filename = data.split(b"#", 2)
                chat_window.info_text.config(state='normal')
                filename=filename.decode()
                chat_window.info_text.insert(tk.END, '[Newfile]: %s\n' % filename)
                chat_window.info_text.config(state='disabled')
                chat_window.info_text.see(tk.END)
                continue
            elif data.decode() == '[CHAT]BEGIN':
                chat_window.add_log('[对方] ', data.decode())
                print(data.decode())  # 打印到控制台
            elif data.decode() == '[CHAT]END':
                sock.close()
                break
            else:
                chat_window.add_log('对方 ', data.decode())
                print(data.decode())  # 打印到控制台
    except OSError:
        pass
```
客户端的其他代码部分就是和图形化界面有关，具体的用法可以自行百度（而且也不是我写的啦，俺的舍友们搓的）
### 服务器端:
服务器端通信部分比较多，但整体思路还是一样的。
新建一个socket，监听PORT（这里是21567，换成其他的理论上也可以）
```
HOST = ''
  PORT = 21567
  ADDR = (HOST, PORT)
  tcp = socket(AF_INET, SOCK_STREAM)
  tcp.bind(ADDR)
  tcp.listen(3)
  Users = []
  Addrs = []
```

开两个队列，这是用来存放客户机尚未接收的文件名，客户机点击receive时，把队列里的元素全部出队。
开放连接，等到加入的用户数达到两个以后中断循环，客户机1和客户机2都新建用来连接的线程。

```
def accept_connections():
  user1_files_queue = queue.Queue()  # 新建用户1的文件队列
  user2_files_queue = queue.Queue()  # 新建用户2的文件队列
  
  while len(Users) != 2:
    tcpCli, addr = tcp.accept()
    Users.append(tcpCli)
    Addrs.append(addr)
    user_box.insert(tk.END, "User" + str(len(Users)) + ": " + str(addr) + "\n")
```
启动线程，target函数是下面的trans，trans1的参数是user[0]，然后才是user[1]，因为对于trans1来说user[0]才是自己，user[1]是对方，trans2则反过来。
```
  trans1 = threading.Thread(target=trans, args=(Users[0], Users[1], 1024, user1_files_queue, user2_files_queue))
  trans1.start()
  trans2 = threading.Thread(target=trans, args=(Users[1], Users[0], 1024, user2_files_queue, user1_files_queue))
  trans2.start()
```
trans函数里的sock1并不一定是user[0]，取决于谁调用了trans。这个函数的功能是不断从sock1中获取消息，判断是否为文本或者文件，是文本则转发给sock2，是文件则存储并进入sock2的待接收队列。

可能有朋友对于涉及到线程的部分有点晕我们可以这么理解：

对于0号客户机，我们开了一个线程，这个线程会一直持续工作，工作的内容是从1号客户机接收消息，再转发给自己。对于1号客户机，我们也开了一个线程，同样的从对方接受消息然后转发给自己。

这个部分可以整合成一个函数trans，并用线程并行地去调用。不能写两个函数直接分别调用是因为这样就会有先后顺序，会导致一方一直在等待发消息，另一方的函数一直运行不了，所以要用线程库保证能并行。

```
def trans(sock1, sock2, BUFSIZ, user1_files_queue, user2_files_queue):
  while True:
    try:
      data = sock1.recv(BUFSIZ)
    except OSError:
      break
    if not data:
      sock1.close()
    else:
      # 判断是否是文件传输标识
      if data.startswith(b"#fileData#"):
        _, filename, filepath, filedata = data.split(b"#", 3)
        save_file(filepath.decode(), filedata)
        message_display.insert(tk.END, f"File received: {filepath.decode()}\n")
        user2_files_queue.put(filepath.decode())
      elif data == b"#receiveAll#":
        send_all_files(sock1, user1_files_queue)
      else:
        try:
          sock2.send(data)
        except OSError:
          sock1.close()
          break
```

## 待优化的地方：
* 只能实现小文件传输，因为原理是一次性把数据全部发过去，TCP连接的上限是64KB，对于更大的文件要分片多次发送，还没有实现这个逻辑。

* 没有离线传输，要实现的话就要用数据库去缓存待发出的消息，以及登记用户的名字、IP等等，相当于多搞一个注册账户的功能，再加一个数据库

* 只能两个人之间传输，要优化的话其实比较好处理，在原来的基础上加一个选择聊天室的模块，把可供连接的用户数增加，不再用trans函数来sock1和sock2这样对话，而是直接服务器给所有客户机广播。

## 多说几句：
第一次拿ptyhon写非人工智能领域的东西，感慨python真好用。
写博客的时候突然好奇真实的聊天场景应该是什么，就去搜了一下QQ，大概原理分发消息和发视频、发文件
* 打视频直接P2P，因为要经过服务器中转的话不仅速度变慢，也给服务器造成很大压力。所以我猜测一定程度上视频比文本更不容易被监管。

* 发文本消息分在线和离线。在线直接P2P传输，速度快，非常及时。如果对方不在线的话，就缓存到数据库，等上线了再发送。你可能会想那P2P不经过服务器的话，腾讯是怎么知道我们聊天内容的？本地会存储聊天消息，我们一边聊天，后台一边向服务器同步消息。这样既能保证聊天很及时，而同步消息没有那么着急，也可以慢慢来。

* 发文件的原理和我上面代码实现的其实很像，也是经由服务器，先储存在服务器，用户接收时再下载。所以qq发文件会有一个七天缓存，服务器只负责保存七天，七天后直接清空。

* 发送的消息可以拿md5或者sha256加密一下，一点都不加的话随便一抓包就知道我们在聊什么了哈哈哈 

## 源码：
### 客户端
```
import os.path
from socket import *
import threading
from time import ctime
from queue import Queue
import tkinter as tk
from tkinter.filedialog import askopenfilename

class ChatWindow:
    def __init__(self, master):
        self.master = master
        master.title('501密聊')
        master.geometry('620x400')  # 设置窗口大小
        self.paned_window = tk.PanedWindow(master, orient=tk.HORIZONTAL, sashwidth=4, sashrelief=tk.RAISED)
        self.paned_window.pack(expand=True, fill='both')

        # 左侧面板
        self.left_panel = tk.Frame(self.paned_window, width=200, height=400)
        self.left_panel.grid_propagate(False)  # 禁止自动调整大小
        self.paned_window.add(self.left_panel)

        # 在左侧面板中添加显示信息的Text组件
        self.info_text = tk.Text(self.left_panel, state='disabled', width=25, height=20, bg='light yellow')
        self.info_text.pack(expand=True, fill='both')
        self.info_text.tag_configure('tag_me', foreground='green')
        self.info_text.tag_configure('tag_user', foreground='blue')

        #在左侧面板中添加"reiceive"按钮
        self.receive_button = tk.Button(self.left_panel, text="Receive", command=self.receive_all_files)
        self.receive_button.pack(side=tk.BOTTOM, pady=5)
        # 右侧面板
        self.right_panel = tk.Frame(self.paned_window)
        self.right_panel.grid_propagate(False)  # 禁止自动调整大小
        self.paned_window.add(self.right_panel)

        # 在右侧面板中添加聊天窗口的组件
        self.log_text = tk.Text(self.right_panel, state='disabled', width=50, height=20, bg='light blue')
        self.log_text.grid(row=0, column=0, columnspan=2)
        self.send_entry = tk.Text(self.right_panel, width=40, height=5)
        self.send_entry.grid(row=1, column=0)
        self.send_entry.bind('<KeyPress-Return>', self.send_message_on_enter)  # 绑定回车键事件
        self.send_button = tk.Button(self.right_panel, text='Send Text [or Enter]', command=self.send_message)
        self.send_button.grid(row=1, column=1,columnspan=2)
        self.scrollbar = tk.Scrollbar(self.right_panel, orient='vertical', command=self.log_text.yview)
        self.scrollbar.grid(row=0, column=2, sticky='ns')
        self.log_text.config(yscrollcommand=self.scrollbar.set)


        self.message_queue = Queue()  # 创建队列用于存储待发送的消息
        self.send_thread = threading.Thread(target=self.send_messages)  # 创建发送消息的线程
        self.send_thread.start()  # 启动发送消息的线程

        # 设置标签样式
        self.log_text.tag_configure('sent_time_tag', foreground='green')
        self.log_text.tag_configure('recv_time_tag', foreground='blue')
        self.log_text.tag_configure('sent_message', foreground='green')
        self.log_text.tag_configure('recv_message', foreground='blue')

        self.file_entry = tk.Entry(self.right_panel, width=40)
        self.file_entry.grid(row=2, column=0)
        self.select_file_button = tk.Button(self.right_panel, text='Select File', command=self.select_file)
        self.select_file_button.grid(row=2, column=1)
        self.send_file_button = tk.Button(self.right_panel, text='Send File', command=self.send_file)
        self.send_file_button.grid(row=2, column=2)

    def receive_all_files(self):
        self.message_queue.put("#receiveAll#")

    def save_file(filename, filedata):
        with open(filename, "wb") as f:
            f.write(filedata)
    # 在客户端的 process_received_messages 方法中

    def select_file(self):
        filepath = askopenfilename()
        if filepath:
            self.file_entry.delete(0, tk.END)
            self.file_entry.insert(tk.END, filepath)

    def send_file(self):
        filepath = self.file_entry.get()
        if filepath:
            self.message_queue.put("#sendFile#" + filepath)
            self.info_text.config(state='normal')
            self.info_text.insert(tk.END, 'File sent complete: %s\n' % os.path.basename(filepath))
            self.info_text.config(state='disabled')
            self.info_text.see(tk.END)
            self.file_entry.delete(0, tk.END)
        else:
            tk.messagebox.showinfo('Warning', 'Please enter a file path.')

    def send_message(self):
        message = self.send_entry.get('1.0', tk.END)
        self.send_entry.delete('1.0', tk.END)
        if message.strip():
            self.message_queue.put(message.strip())  # 将消息放入队列中
            self.add_log('[Me]', message.strip(), is_sent=True)  # 将消息显示在聊天消息列表框中
            print('[Me]', message.strip())  # 打印到控制台

    def send_message_on_enter(self, event):  # 回车键事件处理函数
        self.send_message()

    def add_log(self, user, message, is_sent=False):
        self.log_text.config(state='normal')
        if is_sent:
            time_tag = '[%s] ' % ctime()
            tag_config = ('sent_time_tag', 'green')
            message_tag = 'sent_message'
        else:
            time_tag = '[%s] ' % ctime()
            tag_config = ('recv_time_tag', 'blue')
            message_tag = 'recv_message'

        self.log_text.insert(tk.END, time_tag, tag_config)
        self.log_text.insert(tk.END, '\n')  # 插入换行符

        if is_sent:
            self.log_text.insert(tk.END, user, message_tag)
        else:
            self.log_text.insert(tk.END, user, message_tag)

        self.log_text.insert(tk.END, '%s\n' % message)
        self.log_text.config(state='disabled')
        self.log_text.see(tk.END)
        print(message)  # 打印到控制台

    def send_messages(self):
        while True:
            message = self.message_queue.get()  # 从队列中获取消息
            if message.startswith("#sendFile#"):
                _, __, filepath = message.split("#")
                path_parts = filepath.split("/")
                # 获取最后一个部分
                last_part = path_parts[-1]
                toFile = "你的服务器上的存储路径" #
                toFile+=last_part
                with open(filepath, "rb") as f:
                    data = f.read()
                    tcpCli.sendall(("#fileData#%s#" % toFile).encode() + data)
                toFile= "#sendFile#"+last_part
                tcpCli.send(toFile.encode())
            else:
                tcpCli.send(message.encode())

def recv(sock, BUFSIZ, chat_window):
    try:
        while True:
            data = sock.recv(BUFSIZ)
            print(data)
            print(data.decode())
            if not data:
                break
            if data.startswith(b"#fileData#"):
                _, filename, filepath, filedata = data.decode().split("#", 3)
                toFile = "你的本地文件存储路径" + filepath.strip() 
                with open(toFile, "wb") as f:
                    f.write(filedata.encode())
                time_tag = '[%s] ' % ctime()
                tag_config = ('sent_time_tag', 'green')
                chat_window.info_text.config(state='normal')
                chat_window.info_text.insert(tk.END, time_tag, tag_config)
                chat_window.info_text.insert(tk.END, '[Download] File download complete: {}\n'.format(filepath))
                chat_window.info_text.config(state='disabled')
                chat_window.info_text.see(tk.END)
                continue
            elif data.startswith(b"#sendFile#"):
                _, __, filename = data.split(b"#", 2)
                chat_window.info_text.config(state='normal')
                filename=filename.decode()
                chat_window.info_text.insert(tk.END, '[Newfile]: %s\n' % filename)
                chat_window.info_text.config(state='disabled')
                chat_window.info_text.see(tk.END)
                continue
            elif data.decode() == '[CHAT]BEGIN':
                chat_window.add_log('[对方] ', data.decode())
                print(data.decode())  # 打印到控制台
            elif data.decode() == '[CHAT]END':
                sock.close()
                break
            else:
                chat_window.add_log('对方 ', data.decode())
                print(data.decode())  # 打印到控制台
    except OSError:
        pass

if __name__ == '__main__':
    HOST = '你的ip地址'
    PORT = 21567
    ADDR = (HOST, PORT)
    BUFSIZ = 65536

    tcpCli = socket(AF_INET, SOCK_STREAM)
    tcpCli.connect(ADDR)

    root = tk.Tk()
    chat_window = ChatWindow(root)
    threading.Thread(target=recv, args=(tcpCli, BUFSIZ, chat_window)).start()  # 创建接收消息的线程并启动
    root.mainloop()

    tcpCli.close()
```

### 服务器端

```
from socket import *
import tkinter as tk
import threading
import os
import queue  # 添加队列模块

def trans(sock1, sock2, BUFSIZ, user1_files_queue, user2_files_queue):
  while True:
    try:
      data = sock1.recv(BUFSIZ)
    except OSError:
      break
    if not data:
      sock1.close()
    else:
      # 判断是否是文件传输标识
      if data.startswith(b"#fileData#"):
        _, filename, filepath, filedata = data.split(b"#", 3)
        save_file(filepath.decode(), filedata)
        message_display.insert(tk.END, f"File received: {filepath.decode()}\n")
        user2_files_queue.put(filepath.decode())
      elif data == b"#receiveAll#":
        send_all_files(sock1, user1_files_queue)
      else:
        try:
          sock2.send(data)
        except OSError:
          sock1.close()
          break

def save_file(filename, filedata):
  with open(filename, "wb") as f:
    f.write(filedata)

def send_all_files(sock, files_queue):
  while not files_queue.empty():
    filepath = files_queue.get()
    filename = os.path.basename(filepath)
    print(filename)
    with open(filepath, "rb") as f:
      data = f.read()
    sock.sendall(("#fileData#%s#" % filename).encode() + data)

def accept_connections():
  user1_files_queue = queue.Queue()  # 新建用户1的文件队列
  user2_files_queue = queue.Queue()  # 新建用户2的文件队列
  
  while len(Users) != 2:
    tcpCli, addr = tcp.accept()
    Users.append(tcpCli)
    Addrs.append(addr)
    user_box.insert(tk.END, "User" + str(len(Users)) + ": " + str(addr) + "\n")

  trans1 = threading.Thread(target=trans, args=(Users[0], Users[1], 1024, user1_files_queue, user2_files_queue))
  trans1.start()
  trans2 = threading.Thread(target=trans, args=(Users[1], Users[0], 1024, user2_files_queue, user1_files_queue))
  trans2.start()

if __name__ == '__main__':
  HOST = ''
  PORT = 21567
  ADDR = (HOST, PORT)
  tcp = socket(AF_INET, SOCK_STREAM)
  tcp.bind(ADDR)
  tcp.listen(3)
  Users = []
  Addrs = []

  root = tk.Tk()
  root.title("Chat Server")
  root.geometry("400x300")
  
  # 创建用户框
  user_box = tk.Listbox(root, width=20)
  user_box.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
  
  # 创建用于显示消息的文本框
  message_display = tk.Text(root)
  message_display.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
  
  # 创建一个线程用于接受连接
  accept_thread = threading.Thread(target=accept_connections)
  accept_thread.start()
  
  root.mainloop()
```
