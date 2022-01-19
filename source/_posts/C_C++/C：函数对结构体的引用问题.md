---
title: 结构体变量作为函数参数调用
date: 2021-08-06 15:39:41
tags: 
- C
categories: 
- C/C++
---

&emsp;&emsp;在学DS的时候，编写函数遇到点小问题，就是关于**结构体变量作为函数参数调用**的，所以我想写一篇来总结一下。

&emsp;&emsp;我们先来声明一些变量：

```c
typedef my_struct
{
    int i;
    double j;
};
```

<!--more-->

## 值传递

&emsp;&emsp;如果直接把结构体变量作为函数参数（形参和实参）时，**形参结构体成员值的改变是不会影响实参结构体的**，我们看一下下面这个这个例子：

```c
void func1(my_struct struct1)
{
    struct1.i = 6;
}
int main()
{
    my_struct struct2;
    struct2.i = 17;
    func1(struct2);
    printf("%d",struct2.i);    
    /*result is 17*/
}
```

&emsp;&emsp;可以看到作为实参的结构体struct2的成员i的值并没有改变，那么**如果我们想改变实参结构体的成员值怎么办**？

## 结构体指针变量和结构体数组

### 结构体数组

&emsp;&emsp;将结构体数组作为函数的参数进行传参，修改后的元素的成员值是可以返回到主调函数的。

&emsp;&emsp;详细的就不写了，不是本文讨论重点，以后遇到不会的话再补。

### 结构体指针变量

&emsp;&emsp;函数的参数传递类似于赋值，直接赋变量的值却不返回，肯定不会改变实参的值；但是，我们把指针作为参数呢？那就不一样了。

&emsp;&emsp;我目前的理解是，用指针作为参数的确还是赋值，只不过赋的是指针的值（也就是地址）。（个人理解，未确定过正确与否）

&emsp;&emsp;于是我们可以这么写：

```c
void func2(my_struct *struct1)
{
    /*注意两种引用的书写方式*/
    struct1->i = 6;    //可解释为指向某一结构体struct2的结构体指针struct1引用struct2的成员i
    (*struct).i=6;
}
int main()
{
    int num;
my_struct *struct2,struct1;
    struct2 = &struct1;            //一定要让指针有个归处(doge)
struct1.i = 17;
    func2(struct2);
printf("%d",struct2->i);    /*result is 6*/
    return 0;
}
```

## 总结：结构体变量作为函数参数传递的3种方法

&emsp;&emsp;我们来做一个小总结，其实将一个结构体变量中的数据传递给另一个函数就3种方法：

### 1、用结构体变量名直接作为参数（值传递）

&emsp;&emsp;这种用法我们一般比较少，代码看值传递那一段。

### 2、用指向结构体变量的指针作为实参，将结构体变量的地址传给形参

&emsp;&emsp;这就是上面说的结构体指针变量那一段，这种方法效率很高，但程序不够直接。**我们还有第三种写法，能够兼顾上面两者的优点**。

### 3、将结构体变量的地址作为实参

&emsp;&emsp;实际上思想和第二种一样，**不同的地方在于传递地址的方式**。这种方法是直接使用取地址符&来获取结构体变量的地址，而不是第二种的做法：创建一个指针，指向结构体，然后传指针。

&emsp;&emsp;下面是代码的具体实现：

```c
void func2(my_struct *struct1)
{
    /*注意两种引用的书写方式*/
    struct1->i = 6;        //可解释为指向某一结构体struct2的结构体指针struct1引用struct2的成员i
    (*struct).i=6;
}
int main()
{
    int num;
    my_struct struct1;
    struct1.i = 17;
    func2(&struct1);        //这里是重点，注意与第二钟方法这里的对比
    printf("%d",struct1.i);    /*result is 6*/
    return 0;
}
```
