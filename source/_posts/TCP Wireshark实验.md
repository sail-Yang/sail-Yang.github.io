---
title: TCP Wireshark实验
date: 2022-12-17 17:09:08
tags:
- TCP
categories:
- 计算机网络
- 运输层协议
banner_img: https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.272t2u67u9es.webp
index_img: https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.272t2u67u9es.webp
excerpt: TCP的wireshark实验，对TCP的格式和机制进行研究
---

 这是《计算机网络自顶向下》（第七版）的配套实验，实验文档在[这里](https://gaia.cs.umass.edu/kurose_ross/wireshark.php)。

# 实验目的

1. 了解TCP的**序列号（SEQ）**和**确认号(ACK)**的作用。
2. 了解TCP连接的**建立**和销毁。
3. 了解TCP的拥塞控制算法
4. 了解TCP的流量控制
5. TCP连接性能的计算方法
6. 了解TCP的报文段结构

# 抓包及相关设置

我们只要按照实验文档的指示，在把[这份txt文件](http://gaia.cs.umass.edu/wireshark-labs/alice.txt)送到[服务器](http://gaia.cs.umass.edu/wireshark-labs/TCP-wireshark-file1.html)之前打开wireshark并开始抓包，然后发送该文件，就可以抓到传输过程中涉及的包。

在抓包的过程中有两点需要注意：

1. 包的序列号在Wireshark中默认是相对序列号，可能会和抓的包中的序号对不起来，因此**推荐改成绝对序列号**。修改方法为`Edit→Preferences→Protocols→TCP→Relative sequence numbers disabled`：

   ![修改默认序列号](https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.22fwszo4hnz4.webp)

2. 还有就是wireshark过滤表达式的写法，我们过滤的条件是主机和服务器之间传输的TCP包，所以**过滤表达式**可以这么写：`tcp and ((ip.dst==128.119.245.12 and ip.src==主机IP地址) || (ip.dst==ip.src==主机IP地址 and ip.src==128.119.245.12))`

不自己抓包也行，作者给的wireshark抓包结果[在这里]( http://gaia.cs.umass.edu/wireshark-
labs/wireshark-traces.zip)（其中的`tcp-ethereal-trace-1`），可以照着这个进行分析。

# TCP基础

1. 启动TCP连接的包的TCP SYN区段的序列号是多少？这个SYN有什么用？

   &emsp;&emsp;当客户端请求建立TCP连接时，SYN字段就会<u>**置1以表示请求建立连接**</u>，序列号是232129012，这是一个随机值。

   ---

   

2. `gaia.cs.umass.edu`发给主机来回复SYN的SYNACK区段的序列号是多少？Acknowledgment的值是多少？`gaia.cs.umass.edu`是如何确定Acknowledgment的值的？这个区段在建立连接中有什么用？

   &emsp;&emsp;SYNACK区段的序列号SEQ = 客户端发来的SEQ+1=232129013，Acknowledgment的值为1，表示<u>**服务器接受到了建立连接的请求并且发送了SYN-ACK的确认包**</u>。

   ---

   

3. 包含“HTTP POST”命令的包的序列号是多少？

      &emsp;&emsp;SEQ = 232129013，`PSH`表示**有数据传输**。

   ---

      

4. 将包含POST命令的TCP包作为第一个包，那马前6个包的的TCP 序列号是多少？发送的时间和接到ACK的时间是什么时候？由此算出的RTT是多少？收到每个ACK后，EstimateRTT是多少？假设第一个EstimateRTT = 第一个包的测量RTT，然后计算后续所有相关值。

   - 第一个是带POST的包：SEQ = 232129013，ACK-SEQ=883061786，LEN=565，RTT=0.023265000 s
   - 第二个：SEQ = 232129578，ACK-SEQ=883061786，LEN=1460
   - 第三个：SEQ = 232136878，ACK-SEQ=883061786，LEN=1147
   - 第四个：SEQ = 232145325，ACK-SEQ=883061786，LEN=892
   - 第五个：SEQ = 232153517，ACK-SEQ=883061786，LEN=892
   - 第六个：SEQ = 232161709，ACK-SEQ=883061786，LEN=892

   &emsp;&emsp;关于RTT就不多写了，根据公式算就行。

   ---

      

5. 前六个TCP包的长度是多少？

      &emsp;&emsp;略，上一个问题中有

   ---

      

6. 所有包中，收到的最小的可用缓冲区空间量是多少？？确实接收器缓冲区空间是否会限制发送方发送TCP包的速率？

      &emsp;&emsp;找window最小值就行，这和流量控制有关。

   ---

      

7. 是否有重传包？检查什么以分辨呢？

      &emsp;&emsp;没有重传，因为**序号在增加**：

   <div align="center"><img src="https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.jz740k1fjz4.webp" alt="序号流图" style="zoom:50%;" /></div>

   ---

      

8. TCP连接的吞吐量怎么算？

   <div align="center"><img src="https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.3g6yz22ane20.webp" alt="吞吐量" style="zoom: 67%;" /><div>

      &emsp;&emsp;$吞吐量=\frac{数据传输大小}{所用时间}=\frac{164090B}{5.297341000s}$

   ---

      