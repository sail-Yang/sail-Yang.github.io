---
title: ubuntu解决apt-get报错问题
date: 2021-08-21 21:29:55
tags:
- Linux
- Ubuntu
- 指令
categories:
- 操作系统
- Linux

---

&emsp;&emsp;最近刚学Linux,在Ubuntu上运行apt-get命令时遇到了点问题，目前我也不知道为什么，所以先把解决方法贴出来。

&emsp;&emsp;下面这个是报错的情况：

![image-20210821213538984](https://gitee.com/sail-Yang/blogiamge/raw/master/img/20210821213547.png)

&emsp;&emsp;貌似是找不到什么文件，不能获得什么锁的，还有就是其他的线程在使用apt-get?。上网搜了一下，就是**因为还有一个线程在使用apt-get进行下载的操作。**

<!--more-->

## 解决方法1

&emsp;&emsp;我使用的是这个方法，就一条命令：

```shell
sudo rm /var/lib/dpkg/lock
```

&emsp;&emsp;其实就是删除这个lock就行了，我试了一下是有用的。如果还不行，下面这些一个一个试：

```shell
sudo rm /var/lib/apt/lists/lock
sudo rm /var/cache/apt/archives/lock
sudo rm /var/lib/dpkg/lock-frontend
sudo dpkg --configure -a
sudo apt update
```

## 解决方法2(未亲测)

&emsp;&emsp;这个方法是后来我看到的，但不知道有没有用。上面说了，报错是因为还有一个线程在使用apt-get,那么我们把这个线程关了不就行了吗？

&emsp;&emsp;我们直接执行下面这个命令：

```shell
sudo killall apt apt-get
```
