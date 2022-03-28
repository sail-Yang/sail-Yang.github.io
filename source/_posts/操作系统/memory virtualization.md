---
title: OSTEP Homework2 - memory virtualization
date: 2022-03-22 10:18:18
tags:
- OS
- homework
categories:
- 操作系统
- OSTEP
cover: https://s2.loli.net/2022/03/26/LRJQDmcyX9vZe1E.png
---

&emsp;&emsp;这是内存虚拟化部分的作业。

# 第14章

![](https://s2.loli.net/2022/03/26/FMp1uoaZzOXKrG4.png)

 &emsp;&emsp;首先程序如下：

```c
int main()
{
    int *ptr = null;
    free(ptr);
    return 0;
}
```

&emsp;&emsp;编译后什么事情都没发生。

![](https://s2.loli.net/2022/03/26/ZMAfNo4mYHLePJn.png)

<img src="https://s2.loli.net/2022/03/26/l6Nf5F3qLToZPxD.png" title="" alt="" data-align="center">

![](https://s2.loli.net/2022/03/26/p9vZK1DqFgm34WV.png)

![](https://s2.loli.net/2022/03/26/WyZ8dYQ9HFx1KAp.png)

&emsp;&emsp;额，还是什么都没发生，看来系统很好地解决了内存泄漏的问题。

![](https://s2.loli.net/2022/03/26/o6Cdlf7rnMczv3i.png)

    首先我写的简单程序如下：

```c
int main()
{
    int *ptr = malloc(sizeof(int)*3);
    return 0;
}
```

&emsp;&emsp;GDB还是什么都看不出来：

![](https://s2.loli.net/2022/03/26/OAv2xKpQNglUJSG.png)

&emsp;&emsp;再来看看valgrind怎么样：

![](https://s2.loli.net/2022/03/26/bHWR9FDaerUQjh7.png)

&emsp;&emsp;芜湖，真的检测出来了，我故意泄漏了3个int大小的数据，也就是12 bytes，valgrind检测出来了，并且指出了行号8。

![](https://s2.loli.net/2022/03/26/cwVJMblt2YjHQKo.png)

```c
int main()
{
    int *Arr = malloc(sizeof(int)*100);
    Arr[100] = 0;
    free(Arr);
    return 0;
}
```

&emsp;&emsp;运行时Nothing happened，用GDB运行时：

![](https://s2.loli.net/2022/03/26/5oAqiG7r1NxLSJB.png)

&emsp;&emsp;GDB还是什么都看不出来，我们再拿valgrind看看:

![](https://s2.loli.net/2022/03/26/nkZltcQXCI1Eu69.png)

&emsp;&emsp;可以看到检测出了错误。

![](https://s2.loli.net/2022/03/26/oWB3VfQL6xsZqzc.png)

    GDB还是什么都没检测出来，直接看valgrind：

![](https://s2.loli.net/2022/03/26/S9JCOyhHNQ7uxPB.png)

&emsp;&emsp;检测出了读取错误。
