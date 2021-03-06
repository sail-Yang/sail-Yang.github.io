---
title: vscode使用指南
date: 2021-09-03 14:26:20
tags:
- vscode
categories:
- 工具
cover: https://s2.loli.net/2022/03/26/y5tgnGl7pXUCOzD.png
---

# 前言

&emsp;&emsp;最近在用VScode这款编辑器，为了最大化发挥这个工具的能力，需要记住一些不同的快捷键以及方法，这些玩意需要多用才能熟练掌握，为此，我想写个简易的对照手册，这样能够在我忘记的时候可以快速地查找到想要的命令，提高效率。

<!--more-->

# Vscode杂谈

&emsp;&emsp;Vscode全名为<u>Visual Studio Code</u>,其是微软开发的一款**免费的**、**开源的**跨平台轻量编辑器。

&emsp;&emsp;注意，VScode不是IDE。IDE侧重的是开箱即用的编程体验、对代码往往有很好的智能理解，同时侧重于工程项目，为代码调试、测试、工作流等都有图形化界面的支持，所以比较笨重；而像VScode这种编辑器，**侧重文件夹和文件，支持的语言丰富多样，十分自由**。

&emsp;&emsp;VScode有个很显著的特点就是它是一种开放的平台，VS Code 的源代码以 MIT 协议开源。这不仅意味着大家能够免费获取到 VS Code 的核心代码，更意味着<u>社区能够基于 VS Code 的代码，开发自己的产品</u>，这一点十分重要。就是因为这个特点，我们才会看到vscode上的各种各样的插件，可以实现高度的定制化，打造专属于自己的编辑器。

# 写代码时

&emsp;&emsp;我们首先来讲讲写代码时最常用的操作，熟记这些操作能帮助我们提高编写代码的效率。

## 光标移动

| 作用                 | 快捷键                          |
|:------------------:|:----------------------------:|
| 光标移动到一行（文档）的开头/结尾  | home/end（Ctrl+home/Ctrl+end） |
| 光标移动到当前单词的首和尾      | ctrl+←(ctrl+→)               |
| 回到上一个光标的位置         | Ctrl+U                       |
| 代码块的移动（在一对花括号之间跳转） | Ctrl + Shift + \             |
| 文本选择               | 一直按着shift键，然后按方向键            |

## 删除操作

| 作用            | 快捷键            |
|:-------------:|:--------------:|
| 删除整行          | Ctrl+Shift+K   |
| 删除光标之前的一个英文单词 | Ctrl+backspace |
| 删除光标之后的一个英文单词 | Ctrl+delete    |

## 代码行编辑

| 作用                | 快捷键                          |
|:-----------------:|:----------------------------:|
| 在当前行上（下）快速插入一空行   | Ctrl+Enter(Ctrl+Shift+Enter) |
| 交换上下行的位置          | 按着alt键，然后按方向键                |
| 复制本行代码到插入到下（上）一行  | 按着alt和shift键，然后按方向键          |
| 快速注释一行（整段代码注释）    | Ctrl+/（Alt+Ctrl+/）           |
| 快速跳转到行（比如报错在第n行）  | Ctrl+G+行号                    |
| 跳转到函数定义处/跳转到函数实现处 | F12/Ctrl+F12                 |
| 展现谁引用了函数或类        | Shift+F12                    |

## 代码跳转和查看引用

&emsp;&emsp;有时候我们需要查看一个函数的定义是什么样的或者查看一个函数在哪些地方被引用，这个时候我们就需要一些快捷操作：

| 作用          | 快捷键                                     |
|:-----------:|:---------------------------------------:|
| 查看当前函数定义    | (按住ctrl键，然后鼠标左键) 或者 (移动光标到待查看函数，然后按F12) |
| 查看当前函数被引用情况 | shift+F12                               |

# 文件之间

&emsp;&emsp;有时候我们需要在不同的文件之间切换以及进行一些操作，下面讲讲关于这些的操作。

## 文件级操作

| 作用        | 快捷键                                         |
|:---------:|:-------------------------------------------:|
| 快速打开文件    | Ctrl+P进入文件查找列表（**如果按下ctrl+Enter会在新的编辑器打开**） |
| 快速打开最近文件  | Ctrl+R                                      |
| 快速打开文件第n行 | Ctrl+g，输入行号                                 |

## 全局快捷键

| 作用        | 快捷键                     |
|:---------:|:-----------------------:|
| 打开/关闭侧边栏  | Ctrl+B                  |
| 打开多个编辑器   | Ctrl+\                  |
| 聚焦到第n个编辑器 | Ctrl+n(数字)              |
| 放大/缩小     | Ctrl + +/-              |
| 进入专注模式    | 进：Ctrl+K再按M,退：Esc twice |
