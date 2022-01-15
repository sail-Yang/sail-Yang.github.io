---
title: Linux的ctrl+快捷键的功能
date: 2021-10-02 12:41:13
tags:
- Linux
- Ubuntu
- ctrl
categories:
- 操作系统
- Linux
---

# 前言

&emsp;&emsp;额，最近在看《C++primer》，遇到一个很简单的循环输入：

```c++
while(std::cin >> value)
{
    sum += value 
}
```

&emsp;&emsp;在ubuntu上执行时，结束输入的条件是无效输入或者是文件中止符EOF。Windows的文件中终止符是`Ctrl + Z`,那么Linux系统中是啥呢？这就是这篇总结的源头。于是我顺藤摸瓜，把Ctrl+快捷键都一遍，来这做个总结。

<!--more-->

&emsp;&emsp;首先说明一下，在Linux中可以用`stty -a`命令查看终端的快捷键配置，效果如下：

![image-20211002125045305](https://gitee.com/sail-Yang/blogiamge/raw/master/img/20211002125053.png)

&emsp;&emsp;关于stty的详细内容，下面会说。

# 默认的ctrl快捷键

## Ctrl +Z

&emsp;&emsp;这玩意干什么用的呢？举个例子，我现在运行上面说到的那个小片段，输入一些数后：

<img src="https://gitee.com/sail-Yang/blogiamge/raw/master/img/20211002131113.png" alt="image-20211002131113071" style="zoom:80%;" />

我现在想去做别的事，不想输入了，就是玩儿，怎么办呢？我可以用`Ctrl+Z`，功能是susp，即suspension，停止当前运行的程序，挂起这个程序：

![image-20211002131411566](https://gitee.com/sail-Yang/blogiamge/raw/master/img/20211002131411.png)

等我想继续执行这个程序的时候呢，输入`fg`，就可以回到这个程序：

<img src="https://gitee.com/sail-Yang/blogiamge/raw/master/img/20211002131551.png" alt="image-20211002131551851"  />

其实`Ctrl Z`就是把当前这个进程挂到后台，我们可以用`fg %编号`来把这个进程重新召回到前台来，比如说我们输入`fg %2`就是把编号为2的进程召回到前台：

![image-20211002131920966](https://gitee.com/sail-Yang/blogiamge/raw/master/img/20211002131921.png)

`fg`既然是召回前台，那么有没有后台命令呢？当然有，后台就是`bg`，我们可以用`bg`命令查看后台的情况：

<img src="https://gitee.com/sail-Yang/blogiamge/raw/master/img/20211002132225.png" alt="image-20211002132225146" style="zoom:80%;" />

## Ctrl + C

&emsp;&emsp;`Ctrl Z`是挂起进程，这`Ctrl c`其实就是强制终止正在进成的进程，这没什么好说的，就跟强行关机是一个意思。

## Ctrl+D

&emsp;&emsp;这个就是我之前要的EOF，文件终止符，我们在写程序的时候也会用到这玩意。

&emsp;&emsp;我们来看一个演示，我现在要计算1~10的和，已经把数字输好了：

![image-20211002132722906](https://gitee.com/sail-Yang/blogiamge/raw/master/img/20211002132723.png)

&emsp;&emsp;请问我现在如何<font color=purple>**优雅地结束输入？**</font>按下`ctrl D`就行了。

![image-20211002133025989](https://gitee.com/sail-Yang/blogiamge/raw/master/img/20211002133026.png)

&emsp;&emsp;其实`ctrl D`就发送了一个`exit`信号，我们可以用它来退出root权限。比如我现在登入root:

![image-20211002133243103](https://gitee.com/sail-Yang/blogiamge/raw/master/img/20211002133243.png)

&emsp;&emsp;按下`ctrl D`，就会发送exit，然后退出root，我们可以把这个当作退出快捷键:

![image-20211002133452812](https://gitee.com/sail-Yang/blogiamge/raw/master/img/20211002133452.png)

## 其它的ctrl快捷键

| 快捷键(ctrl用^表示) | 功能            |
|:-------------:|:-------------:|
| ^l            | 相当于clear,清空终端 |
| ^s            | 中断输出          |
| ^q            | 继续输出          |
| \^a/\^e       | 光标移到行首/行尾巴    |
| ^p            | 重复上次命令        |
| ^y            | 粘贴或者恢复上次删除    |
| \^s/\^q       | 冻结shell/解冻    |

# stty

> `stty`命令主要用于查看或修改终端快捷键。

&emsp;&emsp;如果想查看目前终端快捷键的配置，可以使用`stty -a`：

<img src="https://gitee.com/sail-Yang/blogiamge/raw/master/img/20211002134835.png" alt="image-20211002134835492" style="zoom:80%;" />

&emsp;&emsp;一个表达式等号左边是快捷键对应的命令，等号右边是快捷键，举个例子：`eof = ^D`的意思就是`ctrl D`会执行`eof`命令。那么，我们怎么修改命令呢？可以使用 `stty 命令名 快捷键`来更改。********
