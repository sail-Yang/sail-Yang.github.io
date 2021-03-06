---
title: OSTEP Homework1 - CPU virtualization
date: 2022-02-22 17:55:53
tags:
- OS
- homework
categories:
- 操作系统
- OSTEP
cover: https://s2.loli.net/2022/03/26/LRJQDmcyX9vZe1E.png
---

&emsp;&emsp;最近开始系统地学习操作系统原理了，我选择的主要教材是《[操作系统导论](https://book.douban.com/subject/33463930/)》（[英文版在线地址](https://pages.cs.wisc.edu/~remzi/OSTEP/)）。为了方便回顾以及督促自己完成每章的作业，我准备记录下每次完成的作业。由于这本书的组织结构是根据操作系统<u>虚拟化</u>、<u>并发</u>和<u>持久化</u>这三个特性来组织的，而每一种特性又分成几部分来说。这一篇是对CPU虚拟化部分作业的记录，会随着学习进度不断更新。

> &emsp;&emsp;作业的网络资源：[Homework](https://pages.cs.wisc.edu/~remzi/OSTEP/Homework/homework.html)

# 第四章：Process Intro

&emsp;&emsp;这一章主要是简单介绍了进程(**process**)及其相关概念,这一章的作业主要是运行一个模拟器程序——`process-run.py`，具体的操作看README文件就行。

&emsp;&emsp;这一章作业的题目如下：

![](https://s2.loli.net/2022/03/26/Vmh4nIzo5yE3NXc.png)

---

1. 第一题就不说了。

2. 我们先来看看`process list`:

![](https://s2.loli.net/2022/03/26/JiHWfOe2IzYmdCb.png)

&emsp;&emsp;进程`Process0`需要4个时间单元，`Process1`需要7个时间单元(`io`+5个时间单元用来处理+`io_done`)，由于是简单的发出I/O，然后等待完成，所以总时间就是11个时间单元。我们添加参数`-c`来看下结果:

<img title="" src="https://s2.loli.net/2022/03/26/BI9XLcYHUrSFD3z.png" alt="" width="660">

3. 交换一下顺序，我们查看一下现在的process list:

![](https://s2.loli.net/2022/03/26/56R2rHeTywYOWL3.png)

&emsp;&emsp;看上去也就是顺序变了一下，貌似没什么变化，但真的是这样吗？我们看看运行情况：

<img title="" src="https://s2.loli.net/2022/03/26/RMcbtrLiId7f6JG.png" alt="" width="656">

&emsp;&emsp;显然这种情况节省了很多时间，关键就在于**利用了IO设备在工作时(`WAITING状态`)CPU的闲置时间**。

4. 这个问题主要是帮助我们了解-S这个标志的功能，也就是**进程切换时，系统不同行为的影响**。这里我们输入题目指定的指令，查看结果：

![](https://s2.loli.net/2022/03/26/WaJZiFSRYMwQTOr.png)

&emsp;&emsp;我们设置标志为`SWICH_ON_END`，也就是直到一个进程结束了才会切换到另外一个进程。从上图可以看出的确如此，IO进程执行完了才会执行原来的进程。

5. 现在把标志设置为`SWITCH_ON_IO`，也就**是问题3这种情况，在等待IO操作（WAITING）时切换进程**，我们看看结果：

![20220222191321.png](https://s2.loli.net/2022/03/26/IzXLsh4bryqaOWJ.png)

6. 这个问题又介绍了**与IO结束时行为相关**的参数`-I`,我们看看`IO_RUN_LATER`会造成什么结果：

![20220222191907.png](https://s2.loli.net/2022/03/26/45oVIOqyKR7HNmd.png)

&emsp;&emsp;`IO_RUN_LATER`导致IO这个进程就一直挂着，然后运行原来的进程。**我们可以看到资源肯定没有高效利用，17之后CPU的资源基本上全浪费掉了**。

7. 换成`IO_RUN_IMMEDIATE`后我们再来看看结果:

![20220222192430.png](https://s2.loli.net/2022/03/26/xIQ1ShMETbnWL4A.png)

&emsp;&emsp;对比一下问题6，就知道这种方式的效率之高，CPU的Busy Time都`100%`了。

8. 这个问题也就不多说了。

---

# 第五章：进程API

    这一章主要讲了三个创建进程的API：`fork`,`exec`和`wait`。下面是题目：

![](https://s2.loli.net/2022/03/26/SFTtoDw2PdjyhHm.png)

![20220228122422.png](https://s2.loli.net/2022/03/26/Xb94SMqzk8ilIax.png)

---

1. 第一题的代码如下：

```c
#include <stdio.h>
#include <stdlib.h>
#include <wait.h>
#include <unistd.h>

int main()
{
    int x = 100;
    int rc = fork();
    if(rc < 0){
        printf("failed\n");
        exit(1);
    }else if(rc==0)
    {
        x = 70;
        printf("the x is %d (pid:%d)\n",x,(int) getpid());
    }else
    {
        int wc = wait(NULL);
        x = 80;
        printf("the x is %d (pid:%d)\n",x,(int) getpid());
    }
    return 0;
}
```

&emsp;&emsp;结果是显而易见的,**子进程修改变x，并不会影响主进程中的x**：

<img src="https://s2.loli.net/2022/03/26/yk7cjRqGIelmFXD.png" title="" alt="20220228122712.png" data-align="center">

2. 为了查看子父进程是否都能读取成功以及写入成功，我设计了下面这个程序:

```c
#include <stdio.h>
#include <stdlib.h>
#include <wait.h>
#include <unistd.h>
#include <sys/types.h>    
#include <sys/stat.h>    
#include <fcntl.h>
#include <string.h>

int main()
{
    int fd = open("./data.txt",O_RDWR,S_IRWXG);
    // 如果fd == -1,这个时候说明文件打开失败，调用perror查看失败原因
    if(fd == -1)
    {
        perror("Error");
        exit(1);
    }
    int rc = fork();
    if(rc < 0)
    {
        fprintf(stderr,"Fork failed\n");
        close(fd);
        exit(1);
    }else if(rc==0)
    {
        //如果子进程不能读取文件中的内容，就会显示Reading failed
        char chi_str[30] = "Reading failed.\n";
        read(fd,chi_str,30);
        printf("Child process's fd is %d , the result of read : %s\n",fd,chi_str);
        //尝试写入
        char *child_str = "Hello , I am child process.\n";
        write(fd,child_str,strlen(child_str));
    }else
    {
        //如果父进程可以读取文件中内容，就会显示“Reading successfully”；失败同上
        char par_str[30] = "Reading failed.\n";
        read(fd,par_str,30);
        printf("Parent process's fd is %d , the result of read : %s\n",fd,par_str);
        char *parent_str = "Hello , I am parent process.\n";
        //尝试写入
        write(fd,parent_str,strlen(parent_str));
        close(fd);
    }
    return 0;
}
```

&emsp;&emsp;首先来看看程序的运行结果:

![](https://s2.loli.net/2022/03/26/tTGHrD8vm4J2OcB.png)

&emsp;&emsp;虽然子父进程都能访问文件描述符`fd`,但是只有父进程成功读取了文件中的内容。

&emsp;&emsp;我们再来对比一下程序执行前后**的`data.txt`文件**:

![](https://s2.loli.net/2022/03/26/UGjyb2RFxgLMPzA.png)

![](https://s2.loli.net/2022/03/26/fENk5KDX3cxluGJ.png)

&emsp;&emsp;由此可见，子父进程都成功进行了写入操作。

3. 我苦思冥想，实在是找不到只用fork不用wait就能实现子进程先执行的方法，查阅了一下才知道我是吃了没有文化的亏😥。

&emsp;&emsp;我们只要使用`vfork`就行了:

```c
#include <stdio.h>
#include <stdlib.h>
#include <wait.h>
#include <unistd.h>
int main()
{
    int rc = vfork();
    if(rc < 0)
    {
        fprintf(stderr,"Fork failed\n");
        exit(1);
    }else if(rc==0)
    {
        printf("hello\n");
        exit(1);
    }else
    {
        printf("goodbye\n");
    }
    return 0;
}
```

&emsp;&emsp;在这里我先不提vfork和fork的区别，我们直接看`man`里的简短说明：

<img title="" src="https://s2.loli.net/2022/03/26/9rMXDu3szxjEiY4.png" alt="" data-align="center">

&emsp;&emsp;还有根据下面这条知道要在`printf("hello\n")`后面加个`exit(1)`来退出子进程：

![](https://s2.loli.net/2022/03/26/xfokLH8Dm7lzJTC.png)

4. 看[这篇博客](https://www.cnblogs.com/Vivian517/p/7707480.html#2)就知道了，我认为讲的很详细了。如果想原汁原味点的，直接去看`man`

5. 我们先来看看在父进程中使用`wait(NULL)`,`wait(NULL)`会返回什么：

<img title="" src="https://s2.loli.net/2022/03/26/kwdqvsU1Cc7RYSb.png" alt="" data-align="center">

&emsp;&emsp;显然，`wait(NULL)`返回了子进程的PID，如果我们把`wait(NULL)`放入子进程中：

<img title="" src="https://s2.loli.net/2022/03/26/mlQ25A93jVXUBdy.png" alt="" data-align="center">

&emsp;&emsp;wait()返回了-1，之所以会这样是因为11568这个进程没有子进程。

6. 总的来说`waitpid(pid,*status,options)`就是`wait(*status)`的进阶版本，因为其比`wait()`多了两个参数所以更加灵活，具体怎么个灵活法看手册😁，或者看[这个](https://blog.csdn.net/yiyi__baby/article/details/45539993?ops_request_misc=&request_id=&biz_id=102&utm_term=waitpid&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-45539993.nonecase&spm=1018.2226.3001.4187)也行。

7. 这个直接照着他说的做就行：

```c
#include <stdio.h>
#include <stdlib.h>
#include <wait.h>
#include <unistd.h>

int main()
{

    int rc = fork();
    if(rc < 0)
    {
        fprintf(stderr,"Fork failed\n");
        exit(1);
    }else if(rc==0)
    {
        close(STDOUT_FILENO);
        printf("print successfully!");
    }else
    {
    }
    return 0;
}
```

&emsp;&emsp;结果就是什么都没有。我们再来测试一下对父进程有没有影响（加个wait以保证子进程先进行）:

```c
#include <stdio.h>
#include <stdlib.h>
#include <wait.h>
#include <unistd.h>

int main()
{

    int rc = fork();
    if(rc < 0)
    {
        fprintf(stderr,"Fork failed\n");
        exit(1);
    }else if(rc==0)
    {
        close(STDOUT_FILENO);
        printf("print successfully!");
    }else
    {
        int wc = wait(NULL);
        printf("print successfully!");
    }
    return 0;
}
```

![](https://s2.loli.net/2022/03/26/f4Y35dGzs1JnD8o.png)

&emsp;&emsp;并没有什么影响。

8.

```c
   #include<stdio.h>
   #include<unistd.h>
   #include<stdlib.h>
   #include<string.h>
```

```c
   int main() {
    int pipe1[2] = {0,0};
    int p = pipe(pipe1);
    if(p, 0){
        fprintf(stderr,"pipe failed\n");
        exit(1);
    }
    pid_t pid_a = fork();
           if(pid_a < 0)
    {
        fprintf(stderr,"pipe failed\n");
        exit(1);
    }else if(pid_a == 0)
    {
        close(pipe1[0]);
        char *str1 = "hello";
        write(pipe1[1],str1,strlen(str1));
        close(pipe1[1]);
        printf("I am process %d,I say hello to you!\n",(int) getpid());
        exit(1);
    }
    pid_t pid_b = fork();
           if(pid_b < 0)
    {
        fprintf(stderr,"pipe failed\n");
        exit(1);
    }else if(pid_b == 0)
    {
        waitpid(pid_a,NULL,0);
        close(pipe1[1]);
        char str2[10];
        int len = read(pipe1[0],str2,10);
        str2[len] = '\0';
        printf("I am process %d,I receive %s from you!\n",(int) getpid(),str2);
        exit(1);
    }
    waitpid(pid_a,NULL,0);
    waitpid(pid_b,NULL,0);
    return 0;
   }
```

![](https://s2.loli.net/2022/03/26/w8gMNeZ6yk9poGt.png)

---

# 第六章：机制：受限直接执行

  这一章的作业是测量作业，也就是让我们去测量操作系统或者硬件性能的某些方面。本章作业的目标就两个：

- 测量系统调用的成本

- 测量context switch的成本

    题目描述中的大部分内容是给我们的提示。

1. 我们先来讨论一下如何测量系统调用的成本，显而易见的是我们将不断循环对系统的简单调用，然后除以迭代次数就是一个平均的系统调用时间。我们这里使用书中说到的`gettimeofday()`时钟，`systemcall`嘛我们就选择read读取一个空文件:

```c
//ubuntu虚拟机环境下
#include <stdio.h>
#include <stdlib.h>
#include <sys/time.h>
#include <unistd.h>
#include <fcntl.h>

#define INSTANCES 100
#define MAX_LEN 20
int main(int argc,char *argv[])
{
    // 这个很有意思，就是让用户输入测试用例
    long user_instances = argc > 1 ? strtol(argv[1],NULL,MAX_LEN) : 0;
    user_instances = (user_instances ? user_instances : INSTANCES);

    struct timeval start,end;
    int file;char buf;
    unsigned long total_time = 0;
    for(int i=0;i < user_instances ;i++)
    {
        file = open("empty.output",O_RDONLY|O_CREAT);
        if(file == -1)
        {
            fprintf(stderr,"Reading failed\n");
            exit(EXIT_FAILURE);
        }

        gettimeofday(&start,NULL);
        read(file,&buf,0);
        gettimeofday(&end,NULL);
        total_time += (end.tv_usec - start.tv_usec);
        close(file);
    }
    printf("The average time for a sys_call (read 0 byte) is %.3f ms\n",(double) total_time/(double) user_instances)
    return 0;
}
```

![](https://s2.loli.net/2022/03/26/Frxjt7U9oVY5I2g.png)

2. 首先咱要确保第二个实验是在单CPU单核下运行的，这点在虚拟机上很容易实现，如果不是虚拟机，也可以用书中提到的`sched_setaffinity()调用`.

&emsp;&emsp;同样，作者也把测量的思路告诉我们了，关键是要进行进程之间的通信，`lmbench 基准测试`是使用pipe来实现通信；作者还提到了`socket`（即套接字）来实现通信。

&emsp;&emsp;由于这两种方式我都不会，所以先留个坑在这😫

---

# 第七章：进程调度介绍

&emsp;&emsp;本章介绍了两个重要的时间指标：响应时间和周转时间，以及针对这些指标的一些简单调度算法：FIFO,SJF,STCF,RR

&emsp;&emsp;本章题目如下：

![](https://s2.loli.net/2022/03/26/Frxjt7U9oVY5I2g.png)

1. 第一题很简单，照着做就行了：

&emsp;&emsp;首先是FIFO的模拟：

<img title="" src="https://s2.loli.net/2022/03/26/aTY4lj3szvFJCAu.png" alt="" width="628">

&emsp;&emsp;我们自己最好先算一下，然后再用-c参数进行运行，最后结果如下：

<img title="" src="https://s2.loli.net/2022/03/26/EY3urRJAkbfPGql.png" alt="" width="629">

&emsp;&emsp;由于这里的工作时间长度是一样的，所以SJF和FIFO的结果是一样的。

2. 第二题改变一下时间长度即可，然后做同样的事，下面直接上结果：

&emsp;&emsp;如果是100，200，300的顺序，那么SJF和FIFO是相同的；下图是300，200，100的顺序，可以看出差别：

<img title="" src="https://s2.loli.net/2022/03/26/3UgNaHsrMn1P5oj.png" alt="" width="632">

<img title="" src="https://s2.loli.net/2022/03/26/tG3b1EMoyXRVOUu.png" alt="" width="631">

3. `python scheduler.py -p RR -q 1 -j 3 -l 300,200,100 -c`

4. 只要j1<=j2<=j3<=j4即可。

5. 当工作负载的时间长度相同且量子长度 = 这一时间长度的时候，SJF和RR的响应时间是相同的。

6. 这个可以得出一些数据，然后画个图什么的就知道了。

7. 假设量子长度为T，任务数是N，那么平均响应时间就是: $\frac{T+2T+3T+4T+……+(N-1)T}{N} = \frac{T(N-1)}{2}$

---

# 第八章：MLFQ调度算法

&emsp;&emsp;本章主要讲述了MLFQ这种调度算法，题目如下：

<img title="" src="https://s2.loli.net/2022/03/26/2qnXiDFjohcmft6.png" alt="" width="615" data-align="center">

1. `python mlfq.py -j 2 -n 2 -M 0 -s 100 -c`,参数的意义请看README

2. 如下所示：
   
   1. 图8.2：`python mlfq.py  -l 0,200,0`
   
   2. 图8.3：`python mlfq.py -l 0,200,0:100,20,0`
   
   3. 图8.4：`python mlfq.py -l 0,200,0:0,20,1 -i 10 -S` 
   
   4. 图8.5：`-B 50`
   
   5. 图8.6：`python mlfq.py -l 0,120,0:20,100,9 -S -i 1`，另外一个去掉`-S`
   
   6. 图8.7：`python mlfq.py -l 0,100,0:0,100,0 -Q 10,20,40`

3. 只有一个队列就可以变成调度程序

4. `mlfq.py -S -i 1 -l 0,297,99:0,60,0 -q 100 -n 3 -I -c`
