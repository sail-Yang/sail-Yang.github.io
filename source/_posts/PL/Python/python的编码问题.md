---
title: python读写文件时的编码问题
date: 2022-03-17 11:50:47
tags:
- scraper
- encoding
- python
categories: 
- PL
- Python
cover: https://s2.loli.net/2022/03/26/NGAnLf4tjTdxPrh.png
---

&emsp;&emsp;最近在学爬虫，用python把爬到的html代码写到文件中出现了问题。

&emsp;&emsp;首先来看看代码:

```python
# -*-coding:utf-8 -*-
from urllib.request import urlopen
import requests
from bs4 import BeautifulSoup
headers = {
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36'
}
html = requests.get("https://www.douban.com",headers=headers)
file1 = open("douban.txt","w+")
file1.write(html.text)
file1.close()
```

&emsp;&emsp;代码很简单，就是把豆瓣主页的html保存下来，可是却出现了下列问题：

![](https://s2.loli.net/2022/03/26/rzhAvLcKn5TXpbI.png)

&emsp;&emsp;看到这个错误后，我先检查了`html.text`的**type**是str，而python3的默认编码方式是`utf-8`，说明要写入文件的字节流是没问题的；那就说明**要写入的文件的编码方式有问题**。一开始，我将`file1.write(html.text)`替换为`file1.write("abcdefg")`，发现没什么问题；但是当我把`abcdefg`替换为中文+英文或者全中文时，就出问题了——douban.txt文件里出现了乱码，并且编码方式莫名其妙：

![](https://s2.loli.net/2022/03/26/KSFhyb3Hi12OG46.png)

&emsp;&emsp;我在看了下我爬取的html代码，由于是豆瓣，里面有很多的中文，才会导致错误。具体的原因就是：新建的文件是utf-8，但是一旦读取的内容中有中文，就会是GBK编码（当系统默认编码方式是GBK时），这样自然就会导致`UnicodeEncodeError`。

---

&emsp;**&emsp;不管是读操作还是写操作，只要有中文，open()函数就会使用系统默认的编码，而Windows的默认编码方案一般是GBK，这就很容易导致乱码或者直接报错的问题**。

&emsp;&emsp;再举个例子，假如我在一个空的txt文件夹里写入"我爱中国"四个汉字，然后调用open()和read()，看看结果如何：

![](https://s2.loli.net/2022/03/26/bEKreTcqPBZlHk5.png)

&emsp;&emsp;可以看到和一开始几乎一模一样的错误，只不过这里是读取时的解码错误，已经可以看到这里是用GBK解码的。

&emsp;&emsp;如果我们将"我爱中国"改成"你好世界"，结果python读出的是**浣犲ソ涓栫晫**，这样我们就还原了乱码的情况。通过查询，"你好世界"的utf-8编码是`E4BDA0E5A5BDE4B896E7958C`，而上面这堆乱码**浣犲ソ涓栫晫**的GBK编码为`E4BD A0E5 A5BD E4B8 96E7 958C`（<font color=purple>这里的空格实际上是没有的，只是为了看上去清楚点</font>），**发现没，两者是完全一样的**。这就充分证明了我们的观点。前面之所以会报错也可以解释了，实际上就是有多出来的字符无法用GBK编解码，所以就报错了。

---

&emsp;&emsp;最后来说明一下解决方法：

&emsp;&emsp;我们只要指定open()函数的编码方式为utf-8就可以了：

```python
file1 = open("douban.txt","w+",encoding = 'utf-8')
```
