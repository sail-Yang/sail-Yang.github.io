---
title: 'C中NULL,NUL和null的区别'
date: 2021-08-07 21:07:41
tags:
- C
categories:
- C/C++
---

&emsp;&emsp;之前在看别人代码时，看到别人用了个NUL，我心里纳闷：C语言中空指针不是用NULL表示吗？NUL是什么鬼西东？我把他的代码运行一下，发现根本运行不了……心里在想这家伙是不是少打了一个L🤣，看到他后面都这么写，我心里就产生了疑惑……

<!--more-->

&emsp;&emsp;今天在看书的时候，看到了这么一段话：

<img src="https://gitee.com/sail-Yang/blogiamge/raw/master/img/20210807211918.png" alt="image-20210807211917816" style="zoom:67%;" />

&emsp;&emsp;我擦，还真有NUL这种写法……我又去查了一下，**发现NUL其实就是'\0'的意思😅，其本质是字符**（character）。

&emsp;&emsp;而我们的老熟人NULL呢，其实是宏定义，展开之后呢是**空指针常量**。在C语言中，NULL常常被下面这样定义：

```c
#define NULL    ((void *)0)
```

&emsp;&emsp;也就是说NULL 是一个 void *类型的指针(pointers)。

&emsp;&emsp;注意，NULL和null又是不一样的，null是一种概念的统称，不是C标准里的指针常量或者字符。

---

&emsp;&emsp;最后，再来说一个骚气的隐喻😏：

<font color=purple>**当有人给你发null是什么意思呢？**</font>

&emsp;&emsp;我们知道，万物皆可对象，null在有些语言里就<font color=red>**表示对象还未赋值，对象为空，可以理解为"没有对象"**</font>。

Do you understand?😲
