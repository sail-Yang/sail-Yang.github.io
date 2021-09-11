---
title: powershell基础知识
date: 2021-09-11 16:02:41
tags:
- powershell
- windows
categories:
- 工具使用
- powershell
---

<img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcS0Vs-v110BqzY5lHywvBA7kRVJdJ6iCn5ESY0rpfjramokPTHI0ab2Zoejr2_9e3OWbFY&usqp=CAU" alt="PowerShell" style="zoom:150%;" />

# 前言

最近想学一下powershell,查了很多资料，于是想整合一下这些资料，所以本篇是对链接的一个整合，可能会记录一些我不熟悉但是比较有用的地方。

这里先贴出比较详细的教程：

- [微软官方文档](https://docs.microsoft.com/zh-cn/powershell/scripting/overview?view=powershell-7.1)
- [powershell中文博客](https://www.pstips.net/)

<!--more-->

# 什么是PowerShell

这里直接给出一些介绍的链接：

- [PowerShell官方文档介绍](https://docs.microsoft.com/zh-cn/powershell/scripting/overview?view=powershell-7.1)
- [PowerShell-Wikipedia(要翻墙)](https://zh.wikipedia.org/wiki/PowerShell)

## powershell和powershell core的区别

注意，PowerShell 和PowerShell core是不同的，powershell core是微软发布的powershell的新版本（[下载地址](https://github.com/PowerShell/PowerShell/tags)）。

![左边是powershell core,右边是Powershell](https://gitee.com/sail-Yang/blogiamge/raw/master/img/20210911175705.png)

那么两者有什么具体的区别呢？

- PowerShell core是跨平台的，可以在Windows、Linux和Macos上运行，而作为旧版本的PowerShell只能在Windows上运行。
- PowerShell使用的是 .NET Framework和 .NET Standard ; 作为新版本的PowerShell core使用的是 .NET Core和 .NET Standard 。（什么是.NET框架→[.NET官方文档](https://docs.microsoft.com/zh-cn/dotnet/?view=aspnetcore-5.0)，[.NET简单介绍(csdn)](https://blog.csdn.net/weixin_30435261/article/details/95944269?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163134856016780265456793%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=163134856016780265456793&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-95944269.first_rank_v2_pc_rank_v29&utm_term=%E4%BB%80%E4%B9%88%E6%98%AF.NET&spm=1018.2226.3001.4187)）

现在使用的一般都是powershell core，下面的基础知识的介绍也是依托于powershell core.

# cmdlet和别名

## cmdlet简单介绍

cmdlet 是本机 PowerShell 命令，而不是独立的可执行文件。 cmdlet 收集在 PowerShell 模块中，可按需加载。 可以用任何编译的 .NET 语言或 PowerShell 脚本语言本身来编写 cmdlet。

PowerShell用”动词-名词“的方法来命名cmdlet命令，例如： `Get-Command` cmdlet 用于获取在命令行界面中注册的所有 cmdlet。 动词标识 cmdlet 执行的操作，名词标识该 cmdlet 执行其操作的所在资源。

那么，我们如何查看一个`命令的使用说明`呢？

```powershell
//查找某个命令的作用 → GetHelp name，后面加个-Full就是详细说明,加个-examples就是简略说明
Get-Help Get-VM
//如果我们不知道命令的确切名字，可以使用Get-Command *……*
Get-Command *VM*
```

在命令的详细介绍里，会出现下面这些格式，要能看懂：

<img src="https://gitee.com/sail-Yang/blogiamge/raw/master/img/20210911174724.png" alt="说明格式" style="zoom:80%;" />

以上就是获取cmdlet说明的介绍。

## 别名(Alias)

cmdlet命令的起名很清晰，但是对于熟练的使用者来说，每次都要打一个动词加名词肯定很费劲，所以powershell内部很多命令也有不同的**别名**，例如Get-ChildItem，列出当前的子文件或目录。它有两个别名：ls 和 dir，这两个别名来源于unix 的shell和windows的cmd。

别名有两个作用：

- **继承**：继承unix-shell和windows-cmd
- **方便**：方便用户使用。

那么，我们如何查找别名所指的真是cmdlet命令呢？

```powershell
//查看真名
Get-Alias -name ls
//查看可用的别名，介绍两种方法
dir alias: | where {$_.Definition.Startswith("Get")}	//查找以Get开头命令的可用别名
ls alias: | Group-Object definition | sort -Descending Count
```

# 调用入口的优先级

如果我们输入的命令，powershell识别不了，就会报错：

![can't recognize](https://gitee.com/sail-Yang/blogiamge/raw/master/img/20210911180921.png)

那么，powershell是如何与我们输入的命令想匹配的呢？这是存在一个**优先级顺序**的：

![调用入口的优先级](https://gitee.com/sail-Yang/blogiamge/raw/master/img/20210911181453.png)

