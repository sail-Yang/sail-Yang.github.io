---
title: servlet和jsp
date: 2022-10-15 08:55:26
tags:
 - JavaWeb
categories:
- web开发
- java web
banner_img: https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.1etvdeo7ddy8.webp
index_img: https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.1etvdeo7ddy8.webp
excerpt: 入门javaweb，对servlet和jsp的思考
---

# 前言

最近两周在学习Java Web，从了解Tomcat开始，再到学习Servlet，前端三件套等等，这个过程中我学习到了许多知识，也引发了我的一些思考，我认为有必要将这些思考记录下来，**技术总是会过时的，但技术背后的思想不会那么快过时**，而思考能帮助我理解这些技术背后的思想，因此有必要将这些思考记录下来。

# Tomcat

## web服务简介

目前大部分的web服务是以**TCP/IP+HTTP协议**为基础的，那么这样的一个web服务到底是怎样的工作流程呢？

我们拿B站举例，假设我们在浏览器中点击B站的链接或者在地址栏中输入`https://www.bilibili.com`，浏览器就会封装好一个用于请求的数据包，然后经过网络发送到B站的服务器。服务器会根据网络层协议，比如HTTP协议，解析我们发送的请求数据包，然后将静态资源（html,css等等）和从数据库取出的数据等资源封装到响应数据包中，然后发送回我们的浏览器，也就是客户端，之后我们就能看到B站的主页页面了：

<div align="center"><img src="https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.44vbajds3m60.webp" alt="web服务" style="zoom:67%;" /></div>

以上就是比较典型的web服务过程，当然实际情况要复杂的多，比如在实际地处理客户端（浏览器）数据之前，服务器要和客户端建立连接，这就涉及TCP的三次握手；又比如数据出现差错了怎么办，该怎么校验，怎么解决？又或者是丢包了该怎么办？但这些不是我们在web开发时要考虑的问题，因为这大部分是交给**TCP、HTTP这些网络协议**处理的，在这之上，我们获得的数据还要**经过服务器端程序的一层封装**。

## Tomcat简介

Tomcat就是前文提到的一种服务器端程序，它实际上是一个**http服务器+servlet容器**。

作为http服务器，Tomcat为web开发者封装了应用层及其下层的协议，交给开发者的只是**封装好的Request和Response对象**，这样的好处是使得开发者能够更加专注于业务逻辑。

那么servlet容器又是什么呢？

前文说了web服务的第三步是调用服务器端的处理程序去处理封装好的Request对象，然后返回响应对象。其实这些处理程序就是Servlet，而Servlet容器就是用来管理Servlet的，**Tomcat的主体就是由连接器和不同种类的Servlet容器建立起来的**，其核心架构如下：

<div align="center"><img src="https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.25mfalet1zy8.webp" alt="Tomcat架构" style="zoom:80%;" /></div>

一个web应用可以有多个service，比如支付service，订单处理service等等，而一个service中又有连接器和容器，这个容器就是我们说的**servlet**，它负责处理数据，其结构是像**俄罗斯套娃**一样的；而连接器就负责外部数据和servlet容器的连接，**之前说的封装应用层及其下层协议数据**就是连接器干的活。

![servlet层次结构](https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.nibbm6csbsg.webp)

上图展现了容器的内部结构，一层套一层，大层管理小层，具体细节就不多说了。

# Servlet

Servlet的本质呢就是一个Java接口，对于这个Servlet接口，会有不同的实现，我们所说的Servlet就是这些不同的实现。典型的Servlet继承关系如下：

<div align=center><img src="https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.ehnty66lhp.webp" alt="servlet继承关系" style="zoom:80%;" /></div>

**GenericServlet抽象类**实现了**Servlet接口**，其中核心方法为**init()**、**service()**和**destroy()**，**HttpServlet抽象类**专门用来处理HTTP请求，它继承自**GenericServlet抽象类**，实际上重写了**service()**方法，<font color=purple>**在这个service方法里，我们处理客户端发过来的http请求。**</font>

一个Web服务的主要三部分就是**接受请求→处理请求→响应请求**。因为`处理请求(业务逻辑)`是不同的，所以提取出来交给**Servlet**，由程序员来编写业务逻辑；而其它两部分大同小异，所以交给Tomcat里的**连接器组件**干（**解析和封装请求和响应包**）。

## Servlet生命周期

 说到Servlet就不能不提Servlet的生命周期，一个Servlet的生命周期是**创建**→服务→**销毁**：

- 一个Servlet是在被第一次调用时创建，之后服务无数次。
- 直到服务器关闭后，才销毁Servlet。

这就涉及一个问题，**第一个使用Servlet的用户收到响应的时间就会很长**，因为要等对应服务的Servlet启动嘛；如果系统很小，这当然不是事，但是一旦系统变得复杂了，效率就不是很高；解决办法是<u>我们把常用的servlet在服务器启动的时候，就给它启动</u>。

# JSP

在说JSP之前，我们得知道浏览器上的页面到底是什么东西，实际上一个**静态页面**的主体就是**html + css**，html文件就是给页面上的各种组件布局，css就是定制不同组件的样式，比如颜色啊，位置啊等等，我们的浏览器解析html和css后，就会生成我们看到的页面。所谓的http响应呢，实际上就是服务器发送给客户端浏览器的内容大部分是以html和css为主体的静态资源。

早期的JAVA WEB开发呢，实际上就是美工把html文件交给程序员，然后程序员用一行行out.println()将html语句拼接起来响应给客户端，其思路就是<font color=purple>**用Servlet程序将html代码和处理好的数据拼好再放回去**</font>。<u>同时期的PHP呢，搞了一套**在html中嵌入代码来动态生成html内容的方法**，这样就避免了手动print()一行行html的麻烦</u>。

**JSP**呢，就是sun公司仿照PHP，ASP这些工具的思路创建的东西。JSP文件看上去和HTML文件长得差不多，只不过多了一些java代码和不属于html的标签，举个例子：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<c:set var="user" value="${sessionScope.reader}"></c:set>
<div class="person" >
    <link rel="stylesheet" href="${pageContext.request.contextPath}/resources/css/person.css"/>
<h1>个人信息</h1>
<table >
    <tr>
        <td class="column">用户名</td>
        <td class="value">
            <c:out value="${user.userName}"></c:out>
        </td>
    </tr>
    <tr>
        <td class="column">用户类型</td>
        <td class="value">
            <c:if test="${user.userType=='admin'}" var="result">
                <c:out value="管理员"></c:out>
            </c:if>
            <c:if test="${user.userType=='reader'}" var="result">
                <c:out value="读者"></c:out>
            </c:if>
        </td>
    </tr>
    <tr>
        <td class="column">用户密码</td>
        <td class="value">
      		 <c:out value="${user.passwd}"></c:out>
        </td>
    </tr>
        <c:if test="${user.userType=='reader'}" var="res">
            <tr>
                <td class="column">借阅中数量</td>
                <td class="value">
                    <c:out value="${user.rendingCount}"></c:out>
                </td>
            </tr>
            <tr>
                <td class="column">总借阅数</td>
                <td class="value">
                    <c:out value="${user.havaRendCount}"></c:out>
                </td>
            </tr>
        </c:if>
</table>
</div>
```

这就是一个jsp文件，其实jsp就是由**html+Java代码**组成的，那为什么上面的jsp文件中没有出现java代码呢？这是因为java代码被jsp标签取代了，jsp标签实际上就是封装好的java代码，因此实际上jsp标签还是java代码，只不过推荐使用这种写法。

虽然格式上jsp和html差不多，但实际上两者根本不是一回事，**jsp实际上是一个servlet，是在服务器端发挥作用的，而不是像html在浏览器展示数据的**。你servlet不是每次都要一行行println那些html文件吗？那多麻烦啊是不是，你直接把代码写在html里，由服务器来帮你解析成servlet，这样不就省事多了吗？

所以jsp本质上呢是servlet代码的简便写法，方便我们把动态数据嵌入到静态的html中，具体的servlet代码交给服务器里专门的jsp处理程序生成，其最后返回给客户端的，还是一个html静态资源。

