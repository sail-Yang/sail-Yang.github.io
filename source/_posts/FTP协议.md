---
title: FTP协议实验
date: 2022-11-21 16:09:08
tags:
- 文件共享协议
categories:
- 计算机网络
- 应用层协议
banner_img: https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.1eor1rfpy2e8.webp
index_img: https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.1eor1rfpy2e8.webp
excerpt: FTP协议的小实验，主要是使用pyftpdlib package创建一个FTP Server玩玩
---

> 在阅读下文前，可以先掌握这些内容：[FTP理论知识](https://www.yuque.com/topsail/ta49di/rm4ervmy548ugg7m?#)

# 一个简单的FTP Server

在这节里，我们用python的pyftpdlib package实现一个简单的FTP Server和FTP Client，然后进行一些操作，观察其变化。

用python实现的一个简单FTP Server框架如下所示：

```python
import os
from pyftpdlib.authorizers import DummyAuthorizer
from pyftpdlib.handlers import FTPHandler
from pyftpdlib.servers import FTPServer

def main():
    # 实例化一个虚拟用户
	authorizer = DummyAuthorizer()
    # 添加用户，参数为（用户名,密码,用户目录,权限perm）
	authorizer.add_user('yang','123456',r'E:\\onedrive\\WorkSpace\\Python\\Source\\yang_ftp',perm='elradfmwMT')
	authorizer.add_user('li','123456',r'E:\\onedrive\\WorkSpace\\Python\\Source\\li_ftp',perm='elradfmwMT')
    # 添加匿名用户，only need path
    authorizer.add_anonymous(os.getcwd())
	
    # 初始化FTP句柄
	handler = FTPHandler
	handler.authorizer = authorizer
    
	# 指定高位随机端口范围
    handler.passive_ports = range(20000,20033)
	# string returned when client connects
    # handler.banner = "pyftdlib base ftpd ready"
	
    # 初始化FTP Server ，使其监听0.0.0.0:21
	address = ('0.0.0.0',21)
	server = FTPServer(address,handler)

    # 对连接进行限制
	server.max_cons = 256
	server.max_cons_per_ip = 5

	server.serve_forever()

if __name__ == '__main__':
	main()
```

运行服务器程序后：

![运行FTP Server](https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.5ws0umz6agw0.webp)

我们打开cmd，首先使用`ipconfig`查看本机的IPV4地址(假设本机的IPV4地址为`10.135.8.1`)，然后使用`ftp 10.135.8.1`打开windows的ftp客户端，之后进行登录:

![使用ftp客户端](https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.55741yi6cx80.webp)

之后就可以使用windows上的一些ftp命令去访问ftp服务器，执行一些操作，比如查看文件，下载文件:

![操作ftp命令](https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.7ja920kjnqs.webp)

以上就是简单的一个FTP Server的创建及其与FTP客户端的交互，下面分模块进行一下探究。

## Logging management:  登录管理

登录模块是需要我们在创建FTP Server之前（`serve_forever`）去配置的，我们主要配置`authorizer`，其中authorizer.add_user中有一个比较重要的参数是perm，其指定了用户权限，具体的配置参数见[文档](https://pyftpdlib.readthedocs.io/en/latest/api.html#users)。

## Throttle bandwidth：节制带宽

百度网盘，迅雷这些软件下载的时候都会限速，如果不限速，那么就会出问题，如果有人上传/下载大文件，那么直接把带宽给占了，别人还怎么用。所以，我们的FTP Server需要节制一下带宽，通俗点来说就是限流。

pyftpdlib中主要使用`ThrottleDTPHandler`来节制带宽，其使用方法也是十分简单：

```python
dtp_handler = ThrottledDTPHandler
dtp_handler.read_limit = 30720 	# 30 Kb/sec, 即30 * 1024
dtp_handler.write_limit = 30720  

ftp_handler = FTPHandler
ftp_handler.dtp_handler = dtp_handler
```

我们来验证一下是否真的会节制带宽，假设现在用户yang的目录如下：

![image](https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.32xbm2oc0h40.webp)

现在FTP客户端要从FTP服务器下载`YAPA2.exe`文件,首先我们用没有限流的FTP服务器，FTP客户端的下载情况如下所示：

![image](https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.6jdqaavhous0.webp)

可以看到大约13.5MB的文件，只用了0.04s，速度为374Mbps，下面我们对FTP Server进行限速，限速到**30Kbps**：

![image](https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.5744w41sjc40.webp)

可以看到两者速度的差距之大，由此可证明节制带宽是有效的。那么这是怎么做到的呢？

在pyftpdlib的文档中有这么一段话:

> The basic idea behind `ThrottledDTPHandler` is to wrap sending and receiving in a data counter and temporary “sleep” the data channel so that you burst to no more than x Kb/sec average. When it realizes that more than x Kb in a second are being transmitted it temporary blocks the transfer for a certain number of seconds.

其基本思想在于：<u>将上传/下载的数据封装到一个计数器里，然后让数据流间断性地sleep，即间断地阻塞数据流，以达到不多于 xkbps的速度</u>。

# 异常处理

如果给虚拟用户指定的目录在FTP Server主机上不存在，会发生什么呢？

```python
authorizer.add_user('li','123456',r'E:\\not_exist_dir',perm='elradfmwMT')
```

![ValueError](https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.587wyafhuk00.webp)

我们在这里加个异常处理，没这个目录我就创建一个目录：

```python
directory = "E:\\not_exist_dir"
if not os.path.exists(directory):
    os.mkdir(directory)
authorizer.add_user('li','123456',directory,perm='elradfmwMT')
```

还有个地方容易出错的就是端口占用问题，如果我们用的端口已经有其它进程在用了，那肯定得出错，所以也要加个异常处理:

```python
try:
    authorizer = DummyAuthorizer()
    # 添加用户，参数为（用户名,密码,用户目录,权限perm）
	authorizer.add_user('yang','123456',r'E:\\onedrive\\WorkSpace\\Python\\Source\\yang_ftp',perm='elradfmwMT')
	authorizer.add_user('li','123456',r'E:\\onedrive\\WorkSpace\\Python\\Source\\li_ftp',perm='elradfmwMT')
except OSError as e : 
    if "Address already in use" in e :
        print("port is using...")
```



# FTP Client

我们可以用`ftplib`做一个简单的FTP客户端，这是python自带的一个package。来看一个示例代码：

```python
from ftplib import FTP
import time,tarfile,os

def ftpconnect(host,port,username,password):
	ftp = FTP()
	ftp.connect(host,port)						# 连接ftp server，指定ip和Port
	ftp.login(username,password)		# 登录
	return ftp

def downloadfile(ftp,remotepath,localpath):
    ftp.cmd('xxx/xxx')		#进入远程目录
	bufsize = 1024
	fp = open(localpath,'wb')
	ftp.retrbinary('RETR '+remotepath,fp.write,bufsize) # 接受服务器上文并写入本地文件
	ftp.set_debuglevel(0)							# 关闭调试模式
	fp.close()												# 调用close，单方面关闭连接

def uploadfile(ftp,remotepath,localpath):
	bufsize = 1024
	fp = open(localpath,'rb')
	ftp.storbinary("STOR "+remotepath,fp,bufsize)	# 上传文件
	ftp.set_debuglevel(0)
	fp.close()

if __name__ == '__main__':
	ftp = ftpconnect(host="IP地址",port=21,username="yang",password="123456")
	downloadfile(ftp,"/text.txt",r"E:\\Download\\text.txt")
	ftp.close()
```

上面是一个简单的示例，详细的细节可查阅文档。



# 参考资料

1. [pyftpdlib documentation](https://pyftpdlib.readthedocs.io/en/latest/index.html)
2. [ftplib —Python 3.11.0 documentation](https://docs.python.org/3/library/ftplib.html)