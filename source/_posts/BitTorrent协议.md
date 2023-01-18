---
title: BitTorrent协议简介
date: 2022-10-09 08:55:26
tags:
 - P2P
categories:
- 计算机网络
- 分布式
banner_img: https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.2rrho9v2cfa0.webp
index_img: https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.2rrho9v2cfa0.webp
excerpt:  对BitTorrent协议的简介
---

最近经常会下载一些电影，都是通过一个torrent文件+迅雷下载的，这个torrent文件被称为BT种子，说实话这个名字非常熟悉，中学时经常会听到朋友说起，比如有没有种子什么的😅，但是我一直不太了解这种下载方式。这几天使用这种方式下载的时候，发现和一般通过http协议下载的方式有很大不同，于是我就不禁好奇起来，这种下载方式是如何工作的呢？
# 概念介绍
这种下载方式依托于[BitTorrent](https://en.wikipedia.org/wiki/BitTorrent)协议，BitTorrent是一种internet transfer protocol，同样作为一种transfer protocol，BitTorrent与http,ftp(file transfer protocol)这些协议最大的不同在于**BitTorrent是一种distributed transfer protocol（分布式传输协议），什么意思呢**？
http/FTP protocol都是让用户从一个目标站点的服务器上下载文件，而当用户过多时，就会超出服务器的带宽限制，导致下载出现问题；而BitTorrent将文件分片，每个用户都会下载文件的分片，在下载时也会将自己的文件分片发给其他正在下载的用户，这就形成了一种**图结构**，**点对点式**（[Peer-to-peer](https://en.wikipedia.org/wiki/Peer-to-peer)）的分发文件分片： 

![P2P结构](https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.5dixodre9sk0.webp)

正是由于这一特性，BitTorrent在大文件下载和多人同时下载时有非常好的表现。

# Torrent文件
torrent文件，也就是所谓的BT种子，是我们了解这种下载方式的关键，所以下面就来简单介绍一下这种文件的格式与结构。
## 编码方式：bencoding
torrent文件的编码格式是bencoding,这种编码由ASCII码进行编码，包含以下数据结构：

| **string** | 字符串"hello"在bencoding中就是5:hello，即长度标识+:+字符串 |
| --- | --- |
| **integer** | i开头,e结尾，比如123就可以用i123e表示，456用i456e表示 |
| **List** | l开头,e结尾，比如[132,"hello"]就表示为li123e5:helloe |
| **Dictionary** | d开头,e结尾，key必须是string,value可以是四种数据结构的任意一种，比如{'fruit':5,'meat':8}就可表示为d5:fruiti5e4:meati8ee。除此之外，还要注意，Dictionary必须是有序字典，所有key必须按[字典序](https://zh.m.wikipedia.org/wiki/%E5%AD%97%E5%85%B8%E5%BA%8F)升序排列。 |

虽然Bencoding比用纯二进制编码效率低，但由于结构简单而且不受字节存储顺序影响（所有数字以十进制编码，不存在[大端小端的问题](http://zh.wikipedia.org/wiki/%E5%AD%97%E8%8A%82%E5%BA%8F)），因此具有很好的跨平台性，而且具有较好的灵活性，即使存在故障的字典键，只要将其忽略并更换新的就能兼容补充。
## 文件格式
torrent文件本身是一个字典，类似于Json,里面包含了一些key，那么具体包含哪些key呢？首先我们直接打开一个torrent文件:

![torrent文件](https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.5efiz2romt40.webp)

可以看到除了第一行以外，全都是乱码，说明这是一个二进制文件。我们现在已经知道了torrent文件是用Bencoding方式编码的，通过解析器(留个坑)，就可以将其转化为可读的格式:

```cpp
{
    "announce": "udp://engplus.ru:6969/announce",
    "announce-list": [
        [
            "udp://engplus.ru:6969/announce"
        ],
        [
            "udp://cutiegirl.ru:6969/announce"
        ],
        [
            "udp://bubu.mapfactor.com:6969/announce"
        ],
        [
            "udp://bt2.archive.org:6969/announce"
        ]
    ],
    "created by": "qBittorrent v4.3.91",
    "creation date": "2021-02-12 03:02:32",
    "info_hash": "7cda8188321724d1ce7a0ab0b4f9208023dd1521",
    "info": {
        "length": 4936930690,
        "path": ["BT\u4e16\u754c\u7f51(btsj5.com).txt"],
        "name": "\u91dc\u5c71\u884c.\u7279\u6548\u5b57\u5e55.2016.BD1080P.AAC.x264.CHS.BTSJ5",
        "piece length": 1048576,
        "pieces": [
            "10e72eba2cbdc9d9b9791706fcbfb84e51093c86",
            "d91472ddd5e943caa54f8520fcb35fb106033cf2",
            "214e6470e412215c9cf462ea752adc6a7c90e92f",
            "214e6470e412215c9cf462ea752adc6a7c90e92f",
            "903012943eab4ed903f1bc35079f6075f01da4ff",
            "d118c6af14e0f3e06f8008e712322d06088d9c05",
            "500813dd3b29e0c88e0127a2fdf7851be2472a58",
            "…………"
        ]
    }
}
```
从上面我们可以看出一些torrent的字段，下面进行一个汇总：

<img src="https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.1rwj23d240ww.webp" alt="torrent字段" style="zoom:80%;"  align="center"/>

# BT下载流程
## 简单概述
整个的一个BT下载流程是怎样的呢？

1. 制作Torrent种子文件，制作的过程中要指定发布的文件or文件夹、Tracker URL（即announce字段）、是否启用DHT等等。除此之外，还要将要发布的文件切成块（一般是256KB），用SHA1哈希算法计算每块的哈希值，最后上传到Tracker服务器。
2. BT客户端（如迅雷）得到.torrent文件后，就会根据文件中的announce字段找到Tracker服务器地址（连接方式有UDP也有HTTP），然后客户端就会通过Tracker GET请求得到服务，发送哈希值给Tracker，让其查找。
3. 在找到哈希信息后，Tracker服务器就会反连客户端的IP地址和Port（如果是公网），然后返回正在下载这个文件的所有公网用户的IP地址和Port List,之后会将这个客户端的IP地址和Port保存下来，供其他人使用。
4. BT客户端得到这些其他用户IP后，就可以直接连接到这些IP和端口下载资料了。BT客户端会到所有的用户去寻找自己要下载的东西。BT客户端每找到一个用户就建立一个Socket来下载，所以下载的人越多，速度就越快。
## peer之间的通信
下面来看一下点与点之间是如何建立连接的。在使用BT协议下载时，所有正在下载同一资源的peer都是对等的，两个peer之间相互建立的连接也是对等的，对等节点建立的连接的数据传输方向也是双向的，数据可由任何一端发送到另一端。
首先是TCP层的连接，当一个客户端向其他peer发送一个TCP连接请求，这个请求由以下字段组成：
> 1. pstrlen,固定值为19B
> 2. pstr,值为"BitTorrent protocol"
> 3. reserved，保留8字节字段，一般全设置为0、
> 4. info_hash，必须与请求Tracker时发送的info_hash相同
> 5. peer_id，必须与请求Tracker时发送的peer_id相同

在发送上述握手后，应该会受到相同格式的握手，然后我们就将返回的info_hash和发送的进行匹配，以此来确定是否是同一文件，若没问题，那么就握手成功。握手成功后，就会进行**数据的传输**：

---

1. 提供数据的peer会对下载者发送Bitfield，告诉自己拥有的这个文件的哪些块，这些信息就用Bitfield存储（一个二进制bit数组，数组值为1就表示有这个块）。
2. 初始化状态下，BT客户端对其他的peer都是**choked状态**（阻断）。当下载者准备好了，就会发送**Unchoked消息**，可以进行消息传输。
3. 下载者发送Interested消息，告诉提供数据的peer自己要开始下载了。
4. 下载者发送Request消息，其中Payload包含具体的某个块的信息（序号，begin,length）。
5. 提供数据的peer向下载者发送Piece，即真正的块数据。
---

# DHT网络和磁力链接
## DHT网络
通过上面对BitTorrent的简单介绍，我们发现BT十分依赖于Tracker服务器，如果Tracker服务器全挂彩了，那么BT也就没用了，也没有办法解决这个问题呢？
DHT，即**全称分布式哈希表**(Distributed Hash Table)可以有效地解决这个问题，其示意图如下：

![DHT](https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.569wnzvy9ss0.webp)

DHT是一种分布式存储方法，在不需要服务器的情况下，每个节点负责一个小范围的路由，并负责存储一小部分数据，从而实现整个DHT网络的寻址和存储。在DHT网络中，每一个peer都是一个Node，有自己的哈希值。每个Node即是上传和下载piece的BT下载节点；也是**一个小型Tracker，其会保留自己附近的一部分节点信息**。

实现DHT的一种算法：[Khashmir](https://www.tribler.org/Khashmir/)，简单介绍一下：
DHT的每个节点都保存了一张表，上面记录着自己周围节点的ID，当我们向节点A查询B的信息时，就会检索A的表看看有没有B，有就返回B的信息；**没有就返回距离B最近（用XOR计算)的k个节点，然后继续向着k个节点寻找B**。

## 磁力链接
2009年，作为世界第一Tracker服务商海盗湾，关闭了Tracker服务器，因为涉及侵权问题，随后基于DHT网络的**磁力链接**（Magnet URI scheme）就走上了主流舞台，目前已经成为最流行的下载方式。
例如有这样一串链接：`magnet:?xt=urn:btih:53SWOUDWKG6ORSKTJHHE3QXTIBOGU5WU`

- magnet是协议名，xt是资源定位点，urn:btih是Hash方法
- 后面的一串40位16进制数是本文件内容的Hash值，BT下载客户端就是根据这个Hash值在DHT中定位下载文件。
# 参考资料

[1] [BitTorrent 简介](https://blog.csdn.net/riba2534/article/details/115602512?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165154457716781685369761%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=165154457716781685369761&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-115602512.142%5Ev9%5Epc_search_result_control_group,157%5Ev4%5Econtrol&utm_term=BitTorrent&spm=1018.2226.3001.4187)

[2] [https://en.wikipedia.org/wiki/BitTorrent#Design](https://en.wikipedia.org/wiki/BitTorrent#Design)

[3] [磁力链接 - 维基百科，自由的百科全书](https://zh.m.wikipedia.org/wiki/%E7%A3%81%E5%8A%9B%E9%93%BE%E6%8E%A5)

[4] [XIU2/TrackersListCollection](https://trackerslist.com/#/zh)

# 小结
本章内容主要用于总结一些BitTorrent协议和DHT协议的相关知识，由于还未深入学习计算机网络，一些细节的了解还不是很到位，等我到时候学了计算机网络再来深入学习一下😄，这里先留个坑。
