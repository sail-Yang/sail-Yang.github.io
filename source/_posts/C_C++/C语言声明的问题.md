---
title: C语言声明的问题
date: 2021-08-10 09:03:58
tags:
    - C
categories: 
  - C/C++
---

&emsp;&emsp;这一篇主要是对《C专家编程》中关于C语言声明这一问题的总结笔记。

> C语言声明的语法有时会带来严重的问题。                        ——K&R

<!--more-->

## C语言声明模型如此晦涩的原因

&emsp;&emsp;对于编程语言的语法来说，**歧义是非常忌讳的**，因为它使编译器设计者的工作严重复杂化。而C语言’声明语法‘的歧义性渗透进语言的方方面面，正是由于在**组合类型方面**的笨拙行为，C语言变得十分复杂。下面我们来说说，C语言会变成这样的几个原因：

- type model(类型模型)，在C之前并无广泛应用，BCPL（C的祖先）的数据类型也只有二进制，C先天有缺。

- C出现一种设计哲学：<font color=purple>**要求对象的声明形式与它的使用形式尽可能相似**</font>。

&emsp;&emsp;举个例子：int \*p[3]声明了一个指向int的指针数组，并以\*p[i]这样的表达式引用或使用指针所指向的int数据，所以它的声明形式和使用形式非常相似。这样设计的好处是：<font color=orange>操作符的优先级</font>在声明和使用时是一样的；缺点为它是在一个缺点上进行的改进，<font color=orange>操作符的优先级</font>也是C设计不当和复杂之处。

&emsp;&emsp;“声明的形式和使用的形式相似”这种用法可能是C语言的独创，其他语言并没有采取这种方法。放在上个世纪6、70年代，这种设计都是不合理的，**把两种截然不同的东西做成一个样子真的有必要吗**？

- C声明最大的问题：<font color=purple>**无法以一种人们所习惯的自然方式从左向右阅读一个声明。**</font>特别是const,volatile出现之后，情况更糟了。举个例子：
  
  ```c
  /*涉及指针和const的声明可能会出现几种不同的顺序*/
  const int * i;            /*指针所指的对象是只读的*/
  int const * i;            /*指针所指的对象是只读的*/
  int * const i_jelly;    /*指针是只读的*/
  const int * const i_jam;/*指针和其所指的对象都是只读的*/
  ```

&emsp;&emsp;除此之外，C中风格似声明，但却没有标识符的情况，例如强制类型转换，形式参数声明显得没有必要，我们来看一个例子：

```c
/*强制转换成指向数组的指针*/
char (*j)[20];
j = (char (*)[20]) malloc[20];    //作者所诟病的就是(*)的括号，显得十分多余
```

&emsp;&emsp;这一节是这一章的开篇，作者指出了C语言声明设计不合理的地方，**下面会详细介绍C语言声明相关知识，并且给出面对这种复杂声明的方法。**

---

## 声明的形成

### 声明的基础

&emsp;&emsp;声明器（declarator）就是**标识符以及与它组合在一起的任何指针、函数括号、数组下标**，这是C语言声明的核心，下面贴一张表格来总结一下声明器：

| 数量          | C语言中的名字                  | C语言中出现的形式                                                                 |
| ----------- | ------------------------ | ------------------------------------------------------------------------- |
| 0 or many   | 指针                       | 下列形式之一：1.\*const volatile 2. \*volatile 3. * 4.\*const 5.\*volatile const |
| only  one   | direct declarator（直接声明器） | 标识符 or 标识符[] or 标识符（参数）or (声明器)                                           |
| zero or one | 初始化内容                    | = 初始值                                                                     |

&emsp;&emsp;上面所描述的只是声明的一部分——声明器，声明的基本组成如下：

<img title="" src="https://gitee.com/sail-Yang/blogiamge/raw/master/img/20210810095339.png" alt="image-20210810095331255" style="zoom:50%;" width="467" data-align="center">

&emsp;&emsp;注意，要想声明合法，还得注意：

- 函数的返回值不能是一个函数，所以像foo()()这样是非法的；
- 函数的返回值不能是一个数组，所以像foo()[]这样是非法的；
- 数组里面不能有函数，所以像foo这样是非法的。

&emsp;&emsp;但下面这些是合法的：（**简单来说，指针yyds**）

- 函数的返回值允许是一个函数指针，如int(* fun())()；
- 函数的返回值允许是一个指向数组的指针，如int(* foo())[]；
- 数组里面允许有函数指针，如int (* foo[])()；
- 数组里面允许有其他数组，所以你经常能看到int foo\[]\[]

&emsp;&emsp;在我们看看组合类型是怎么恶心人之前，我们先来复习一下下面这些知识。

---

### 结构、联合和枚举

&emsp;&emsp;额，其实这部分没必要特别写，只是我自己也不太清楚联合、枚举的意思，所以权当复习，会的话可以直接跳过。

#### 联合

&emsp;&emsp;结构是什么，没必要再细说了，就是一种把一些数据项组合在一起的数据结构。所谓的**联合**，其实声明和结构一模一样，只要把struct换成union一样就行了。

> **那么联合和结构有什么区别呢？**
> 
> 在结构中，每个成员依次存储；而**在联合中，所有的成员都从偏移地址零开始存储**。这样，每个成员的位置都重叠在一起：**在某一时刻，只有一个成员真正存储于该地址。**

&emsp;&emsp;联合的存储特性决定了它的几个有限的用途：

- 联合**一般是作为大型结构的一部分存在的**。在有些大型结构中，存在一些与实际表示的数据类型有关的隐式或显式的信息。如果存储数据时是一种类型，但在提取该数据时却成了另外一种类型，这显然存在着明显的类型不安全性
- 节省备用的存储空间。我们可以用联合来存储彼此相互矛盾的数据（即不可能同时出现），比如男和女，你总不可能又是🚹的又是🚺的吧。
- **把同一个数据解释成两种不同的东西**，而不是把两个不同的数据解释为同一种东西。

#### 枚举(enum)

<img src="https://gitee.com/sail-Yang/blogiamge/raw/master/img/20210810101203.png" title="" alt="image-20210810101203170" data-align="center">

&emsp;&emsp;长这样的就是枚举，sizes是可选标签，后面是标识符的列表以及赋给他们的值。在C语言中，#define其实干了enum的活，只不过其它语言都有enum，那咱C语言不能没有(doge)

### 优先级

&emsp;&emsp;读声明得有个先后顺序，这就是优先级，懒得写了，直接看表吧：

<img title="" src="https://gitee.com/sail-Yang/blogiamge/raw/master/img/20210810101650.png" alt="image-20210810101650022" style="zoom:67%;" data-align="center" width="570">

### 读声明的方法

#### 通过图表

&emsp;&emsp;我们来看一个分析的过程，假设我们现在要分析：

```C
char * const *(*next)();
```

&emsp;&emsp;下面是分析过程：

<img title="" src="https://gitee.com/sail-Yang/blogiamge/raw/master/img/20210810102036.png" alt="image-20210810102036341" style="zoom:67%;" width="497" data-align="center">

&emsp;&emsp;拼在一起，读作：“next是一个指向函数的指针，该函数返回另一个指针，该指针指向一个只读的指向char的指针”，

#### typedef

&emsp;&emsp;我们知道typedef的作用是：**它为一种类型引入新的名字，而不是为变量分配空间**。也就是指明一种类型的同义词，而不是去定义一种类型。这毫无疑问，会给我们的声明带来简便这一好处。

&emsp;&emsp;但千万不要像下面这样做：

<img title="" src="https://gitee.com/sail-Yang/blogiamge/raw/master/img/20210810102821.png" alt="image-20210810102821499" style="zoom:67%;" width="598" data-align="center">

##### typedef 和宏文本替换define的区别

&emsp;&emsp;我们应该注意到，typedef的功能和宏文本替换的功能极其相似，那么两者有什么区别呢？

&emsp;&emsp;在说区别之前，我们要有这样的认识：<font color=purple>**typedef是一种彻底的“封装”类型，在声明它之后，就不能再往里面加东西了。**</font>下面来说说区别：

- 可以用其他类型说明符对宏类型名进行扩展，但对typedef所定义的类型名却不能这样做。
  
  ```c
  #define apple int
  unsighed apple i;    /*legal*/
  typedef int peach;
  unsighed peach i;    /*illeagl,你在想桃子吃*/
  ```

- 在连续几个变量的声明中，用typedef定义的类型能够保证声明中所有的变量均为同一种类型，而用#define定义的类型则无法保证。
  
  ```c
  #define int_ptr int *
  int_ptr a,b;        /*我们展开：int * a,b;其实只有a 为int*型，而b只是一个int类型。*/
  #typedef int * int_ptr;
  int_ptr a,b            /*这个时候a,b的类型都是指向int的指针*/
  ```

&emsp;&emsp;这些就是typedef和define的区别。

##### typedef和结构

<img title="" src="https://gitee.com/sail-Yang/blogiamge/raw/master/img/20210810104832.png" alt="image-20210810104832794" style="zoom:67%;" width="561" data-align="center">

&emsp;&emsp;这些就是本篇的内容了。
