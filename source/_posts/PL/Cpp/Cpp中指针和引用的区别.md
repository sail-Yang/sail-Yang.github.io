---
title: C++中指针和引用的区别
date: 2022-02-13 12:36:52
tags: 
- Cpp
- 指针和引用
categories: 
- PL
- Cpp
cover: https://gitee.com/sail-Yang/blogiamge/raw/master/img/20220214122554.png
---

&emsp;&emsp;C和Cpp都有指针，不过Cpp比C多了一个引用(reference)，一般书里都会说**引用是变量的别名（alias）**，但这种定义太过模糊。

&emsp;&emsp;下面就来说说Cpp中的引用到底是什么，以及它和指针有什么区别。

# 引用是什么

     引用的确是已经存在的变量的别名，但别名这种说法太过笼统。**事实上，引用就是一个常量指针**。为了证明这一点，我们进行一个小实验：

&emsp;&emsp;现在有一段代码：

```cpp
int main()
{
    int num1 = 2;
    int *pointer_num1 = &num1;
    int &ref_num1 = num1;
}
```

&emsp;&emsp;下面我们把这段代码转化成汇编代码：

> 我们会使用[godbolt.org](https://godbolt.org/)这个网站来查看C++代码的x86-64汇编代码，这个网站还能查看每句对应的汇编，十分方便。

<img src="https://gitee.com/sail-Yang/blogiamge/raw/master/img/20220213132527.png" title="" alt="" width="670">

&emsp;&emsp;我已经把对应的代码用不同颜色的笔标出来了，可以很清楚地看到指针和引用的行为是一模一样的，这充分说明了指针和引用的基本原理是一样的。

&emsp;&emsp;既然引用和指针一样，那么怎么体现是const指针呢，这汇编代码里也看不出来啊。

&emsp;&emsp;首先要说明，像const这种存取的控制是实现在编译器这一层的，也就是C++底层的机制，在汇编代码这一层是不存在什么const的（我们可以用内存级工具修改任何一个变量）。

&emsp;&emsp;在C++中，**引用一旦绑定初始化对象之后，是不能修改其绑定对象的，这和const指针的行为是一模一样的**。

> &emsp;&emsp;引用只是一种**抽象**，一种概念，具体怎么实现，可能每个编译器都不一样。上面的汇编只是用来了解引用到底是什么东西，但是不能用汇编代码去解释引用！

---

# 引用和普通指针的区别

&emsp;&emsp;上面只是对引用这个概念底层的一点探索，日常使用肯定还是注重引用的特性和使用。下面就对引用和普通指针行为上做一个对比：

![](https://gitee.com/sail-Yang/blogiamge/raw/master/img/20220213135045.png)
