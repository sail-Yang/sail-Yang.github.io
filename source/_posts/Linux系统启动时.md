---
title: Linux系统启动时
date: 2022-10-02 11:44:09
tags:
- 操作系统
- Linux
- 软硬件交互
- BIOS
categories:
- 操作系统
banner_img: https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.2vpyuzqapia0.webp
index_img: https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.2vpyuzqapia0.webp
excerpt: 从我们按下开机键到显示器上显示登录界面，这段时间里到底发生了什么？这篇文章将做一个简单的介绍
---

在Jyy老师的一节OS课上，内容是简单介绍Linux的启动流程，听完后我想更具体点知道这个启动的流程，所以特地搜素了一圈资料，以解答我心中之疑惑。

<div align=center><img src="https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.1zna45ggj5i8.webp" alt="Linux启动流程" style="zoom: 67%;" /></div>

# CPU Reset

启动的第一步就是**Reset CPU**，即<u>设置CPU的各个寄存器为固定、已知的状态</u>。当我们按下电源按钮后，电源就开始给设备供电，一开始供电并不稳定，主板检测到后，就会一直给CPU发送**RESET信号**，此时的CPU就会清除寄存器上的残留数据并进行寄存器的设置：

<div align=center><img src="https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.3gbzr1spgf40.webp" alt="image" style="zoom:67%;" /></div>

在这之中最重要的寄存器是**CS（代码段）**和**EIP（指令指针）**。CS中隐式的Base（基地址）会与EIP相加，即`ffff 0000 H+fff0 H = ffff fff0 H`,其结果地址`0fff fff0 H` 被称为**重置向量**，<font color=purple>**主要包括一条跳转指令，指向BIOS的入口地址**</font>。

<p class="note note-primary">为什么重置向量不直接设为BIOS地址，而是跳转到BIOS呢？</p>

<details>
<summary>跳转到BIOS的解释</summary>
<p><b>注意X86芯片一开始运行在实模式下，实模式只有20位寻址（1M空间），</b>而我们的重置向量是fff0H，到ffffH只有很短的距离，空间很小，不足以放下BIOS程序，所以要跳转。</p>
<p><b>至于为什么不一开始就把重置向量设置为BIOS入口，</b>可能是希望把ROM放到可寻址空间的高层，以留出更大空间给内存。</p>
</details>

<div align=center><img src="https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.4nbqyewemxw0.webp" alt="实模式内存布局" style="zoom: 80%;" />
</div>

<p class="note note-info">多核系统的CPU RESET</p>

<details>
<summary>多核系统下CPU RESET的解释</summary>
    <p>多核计算机下，一开始要执行某个协议，系统将选取一个CPU来执行，其他CPU都等待，这个被选取的主CPU叫<b>Bootstrapping CPU（BSP）</b>。只有主CPU继续执行，其它CPU都进入一个等待状态，等待BSP给它们发指令，它们再开始继续执行。</p>
</details>

# BIOS执行

BIOS是固定在EPROM中的程序，一般由硬件厂商写死，其负责引导系统的启动。其首先执行的程序的就是**加电自检（Power On Self Test，POST）**。

## POST（加电自检）

POST用于计算机刚接通电源时对硬件部分的检测，如果自检中发现不是很严重的错误，系统会根据检测代码**给出提示信息或鸣笛警告**。

**之所以使用鸣笛警告，是因为此时的显卡还没有完成初始化，无法将错误信息显示在屏幕上，只能通过声音报告错误**。

检测的过程是逐一进行的，BIOS厂商对每个设备都提供了一个`POST CODE`，当对某个设备进行检测时，就会把这个POST CODE装到诊断的端口，如果没通过，这个CODE就会被保留，进行报警。

## 初始化设备

POST结束后，BIOS还会调用各设备的BIOS进行自检和初始化，比如显卡BIOS，其会自检和初始化显卡，这个时候显示器就可以显示了。所有设备都是在这个时候被激活的。除了初始化设备，BIOS还会初始化[中断向量表](https://www.cnblogs.com/jadeshu/p/10663505.html)。

## 启动Bootloader

在检测和初始化各种设备之后，BIOS会查找用户自定义的启动顺序，默认是磁盘，当然也可以是光盘和U盘（以前重装操作系统时就会用到光盘和U盘）：

<div align=center><img src="https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.5npilzacmlg0.webp" alt="image" style="zoom:80%;" /></div>

根据顺序，BIOS会把排在最前面的设备的**MBR（主引导记录）**，即<u>第0磁道第一个扇区的512字节</u>，读到**RAM绝对地址0x7C00处**，并跳转到这个地址。

<p class="note note-primary">MBR格式</p>

MBR不属于任何一个操作系统，其512字节的内容如下：

---

- **启动代码(446B)**

&emsp;&emsp;检查分区表的准确性，并将系统控制转移给硬盘上的Bootloader程序(比如现在常用的grub)。

- **磁盘分区表(16 X 4B)**

&emsp;&emsp;DPT，由四个分区表组成，每个16B，说明磁盘的分区情况。

- **结束标志(2B)**

&emsp;&emsp;该标志错误，无法启动。

---

# Bootloader

所谓的Bootloader程序就是用来<font color=purple>**加载操作系统内核文件到内存**</font>，这类程序的主要功能为：

---

1. 从实模式使能保护模式，从16位寻址空间切换到32位寻址空间，使能**段机制**。
2. 从硬盘读取ELF格式的kernel（跟在MBR后面的扇区）并放到内存中固定位置，这个过程一般分两步，最终是**执行`boot`命令**，加载系统引导菜单(`/boot/grub/menu.lst或grub.lst`)，`内核vmlinuz`和`RAM磁盘initrd`。
3. 跳转到操作系统入口点执行，控制权交操作系统。

---

我们这里以GNU的**grub**为例。grub可用于选择操作系统分区上的不同内核，也可用于向这些内核传递启动参数。grub被载入一般包括两个步骤：

---

1. 装载基本的引导装载程序—stage1，其主要功能是装载第二引导程序:stage2。
2. 装载第二引导装载程序——stage2，这第二引导装载程序用于引出更高级的功能， 以允许用户装载入一个特定的操作系统。在grub中，这步是给用户显示一个菜单或是让用户输入命令。**该阶段引导的最终状态就是执行boot命令，将操作系统内核加载进入内存中，进而将控制权转交给内核。**

---

当内核加载完成后，内存被映射为：

<div align=center>
<img src="https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.38oakvdk2ge0.webp" alt="内存映射" style="zoom: 60%;" />
</div>


## UEFI

现在说BIOS，主要指的是UEFI，而不是传统的BIOS。不管是传统BIOS还是UEFI，都是经过ROM→RAM→BOOT的过程，主干是没有区别的，那么我们为什么需要UEFI呢？

可以参考这个回答：[UEFI 引导与 BIOS 引导在原理上有什么区别？](https://www.zhihu.com/question/21672895)

# Linux内核设置和启动

Linux内核启动后，执行的是一系列检测，检测完成后跳转到`start_kernel函数`。这个函数会以依次初始化各个模块，比如页表、中断向量等，然后变成0号进程。0号进程会fork出1号kernel_init进程，`kernel_init`会执行init程序，大体的过程如下：

---

1. init进程读取`/etc/inittab`文件，作用是设定Linux的运行等级，决定进程运行模式。
2. Linux执行第一个用户层文件`/etc/rc.d/rc.sysinit`，该文件功能包括：设定PATH运行变量、设定网络配置、启动swap分区、设定/proc、系统函数等.
3. 读取`/etc/modules.conf`及`/etc/modules.d`目录下的文件来加载系统内核模块。
4. 启动一些服务，之后执行`/etc/rc.d/rc.local`文件。
5. 最后执行/bin/login程序，启动到登录界面，让用户输入用户名和密码。

---

完成用户的启动后，0号进程进入cpu_idle,变成idle进程。到这Linux差不多就启动好了。

# 参考

> 1. http://home.ustc.edu.cn/~hchunhui/linux_boot.html
> 2. [https://437436999.github.io/2020/02/15/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B/](https://437436999.github.io/2020/02/15/操作系统启动过程/)
> 3. https://www.anquanke.com/post/id/227940
