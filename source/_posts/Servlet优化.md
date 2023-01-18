---
title: Servlet优化
date: 2022-11-10 11:44:09
tags:
- JavaWeb
categories:
- web开发
- java web
banner_img: https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.23k31ke5b9a8.webp
index_img: https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.23k31ke5b9a8.webp
excerpt: 记录优化Servlet的过程
---



在学习Javaweb基础的时候，遇到了一个优化Servlet的过程，我觉得有必要记录下优化的过程，这其中体现了一些设计模式和规范。

# 优化前项目的结构和问题

优化前的项目，其基本思路就是<u>服务器接受请求，然后通过路径的映射，找到对应的Servlet来处理请求</u>。比如url路径为`localhost:8080/elibrary/admin?operate=del?id=1`，那么服务器会通过`/admin/del`找到`AdminDelServlet`，然后获取参数`id`，对数据库进行删除操作。大致流程如下图所示：

<div align=center><img src="https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.1hz7bss0nd34.webp" alt="image" style="zoom: 67%;" /></div>

这种结构有一个很突出的问题就是**Servlet太多了**，随着项目的开发，管理难度会越来越大，那么怎么解决呢？

# 优化1：合并增删查改

解决办法也很简单，我们把一个实体对应的操作合并为一个Servlet，比如把`AdminDelServlet`、`AdminAddServlet`、`AdminEditServlet`合并为一个`AdminServlet`，合并后的Servlet根据Switch-Case来选择具体的操作，合并后的示意图如下：

<div align=center><img src="https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.hn7h791qaog.webp" alt="image" style="zoom:67%;" /></div>


代码示例如下：

```java
@WebServlet("/admin")
public class AdminServlet extends HttpServlet{
    @override
    protected void service(HttpservletRequest req,HttpServletResponse resp){
        operate = req.getParameter("operate");
        switch(operate){
            case "del": 
                del(...);
                break;
            case "add":
                add(...);
                break;
            case "edit":
                edit(...);
                break;
        }
        
    }
    //原来的AmdinAddServlet替换成下面的add()
    public void add(...){
        ...
    }
    
    public void del(...){
        ...
    }
    
    public void edit(...){
        ...
    }
}
```

这样写又引出了一个问题：<u>如果一个实体对应的操作很多，那我们的switch-case岂不是得变得特别长</u>？我们能不能优化一下这一部分呢？

我们可以用反射来代替上面的代码，示例代码如下：

```java
//后去AdminServlet的所有方法
Method[] methods = this.getClass.getDeclaredMethods();
for(Method m : methods){
	String methodName = m.getName();
	if(operate.equals(methodName)){
		try{
			m.invoke(this,request,response);
		}
	}
}
```

通过反射机制，我们大大缩减了代码量，不管我们写多少具体操作，这段选择具体操作的代码都不会增多。这样，我们就完成了第一步的优化。

# 优化2 ：引入DispatcherServlet

上节我们使用反射优化了AdminServlet中选择操作的代码，解决了代码量膨胀的问题。与这种情况类似的是，我们的实体也会有很多，比如实际项目中会有许多个类似AdminServlet的Servlet，这其中必然**存在许多冗余的代码**，比如说上节的反射机制选择操作，每个实体Servlet都会有这么一段代码。我们要优化的一个重要部分就是**代码的冗余**，<font color=purple>**基本解决方法就是将冗余的这部分提取出来到一个通用的处理器里**</font>。

我们引入一个`DispatcherServlet`，然后把`AdminServlet`中的反射选择操作提取出来。

我们首先要想办法选择`AdminServlet`，这该怎么办呢？url路径中的`/admin`对应了`AdminServlet`，这是一个映射，<u>我们只需要在一个文件里保留这层映射关系，就能通过java获取到`AdminServlet`的实例</u>。我们创建一个`applicationContext.xml`：

```xml
<?xml version="1.0" encoding="utf-8" ?>
<beans>
    <bean id="admin" class="..." />
</beans>
```

接下来，只要分析xml，就可以得出映射关系，我们将映射关系存到beanMap里:

```java
InputStream inputStream = getClass().getClassLoader().getResourceAsStream("applicationContext.xml");
DocumentBuilderFactory documentBuilderFactory = DocumentBuilderFactory.newInstance();
DocumentBuilder documentBuilder = documentBuilderFactory.newDocumentBuilder();

Document document = documentBuilder.parse(inputStream);

NodeList beanNodeList = document.getElementsByTagName("bean");

for(int i=0; i<beanNodeList.getLength(); i++){
    Node beanNode = beanNodeList.item(i);
    if(beanNode.getNodeType() == Node.ELEMENT_NODE){
        Element beanElement = (Element) beanNode;
        String beanId = beanElement.getAttribute("id");
        String className = beanElement.getAttribute("class");
        Object beanObj = Class.forName(className).newInstance();
        /*将映射关系存到一个beanMap里*/
        beanMap.put(beanId,beanObj);
    }
}
```

得到映射关系后，我们就要想办法从**请求**中得到对应``AdminServlet`的字符串，然后调用AdminServlet方法，这部分思路就很清晰了，直接substring一下url路径就行了:

```java
String servletPath = req.getServletPath();		 //servletPath = "/admin"
servletPath = servletPath.substring(1);				//servletPath = "admin"
Object controllerBeanObj = beanMap.get(servletPath);	//controllerBeanObj = AdminServlet
```

与之相同的道理，我们还可以获取具体操作的字符串，也就是为调用AdminServlet具体的方法做准备：

```java
String operate = req.getParameter("operate");
if(StringUtils.isEmpty(operate)){
	operate = "index";
}
```

获取到实例后，我们就要开始提取**反射选择具体方法的代码**了，我们直接从AdminServlet中提取这部分代码，然后转移到DispatcherServlet中，并且修改为：

```java
/*
url = localhost:8080/porjectname/admin?operate=del&id=2
servletPath = "admin"
operate = "del"
*/
Method[] methods = controllerBeanObj.getClass().getDeclaredMethods();
for(Method method : methods){
	if(operate.equals(method.getName())){
		method.setAccessible(true);
        method.invoke(controllerBeanObj,req,resp);
    }
}
```

# 优化三：提取视图资源处理和获取参数

优化还没结束，我们还是遵循相同的底层逻辑——**提取冗余的代码**，那么还有哪些冗余的代码呢？

1. 获取参数的部分，每个具体操作都要从req中获取不同的参数。
2. 资源的跳转：服务器的内部转发、重定向和交给视图模板处理的代码都是冗余的

## 提取获取参数的代码

我们来看一个具体操作del()的示例:

```java
public class AdminServlet extends HttpServlet{
    service(...){
        /*empty，因为仅存的选择方法的反射代码也被提取到了Dispatchet中*/
    }
    
    public void del(HttpServletRequest req,HttpServletResponse resp){
			String id = req.getParameter("id");
            ...//业务逻辑
             resp.sendRedirect("index");
    }
}
```

我们要做的优化，**就是把getParameter和sendRedirect这两步步放到Dispatcher里**，修改后的del应该是这个样子的:

```java
public class AdminServlet extends HttpServlet{
    service(...){
        /*empty，因为仅存的选择方法的反射代码也被提取到了Dispatchet中*/
    }
    
    public void del(String id){
            ...//业务逻辑
             return "redirect:index"
    }
}
```

我们先来看看如何提取获取参数的代码，基本思路还是<u>利用反射去获取del方法的所有参数名，取得del方法的参数名后去req里获取请求的参数</u>。method刚才已经取得了，获取其参数的类型以及名称只要一句就够了：

```java
Parameter[] parameters = method.getParameters();
```

但是这样取出的参数名，我们会发现是“arg0”,”arg1”,”arg2”这样的，显然不符号我们的要求，因此，我们需要**在java compiler选项里加上`-parameters`**，这样获取的参数数组就是带名字的了:

![带parameters选项的获取参数](https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.5jpd5xrjw480.webp)

获取到参数的名字后，我们就可以使用参数名去获取请求中存储的值了:

```java
Parameter[] parameters = method.getParameters();
/*用于存取参数对应的值*/
Object[] parameterValues = new Object[parameters.length];
for(int i=0;i< parameters.length;i++){
    /*处理参数中有特殊的情况*/
    if(parameters[i].getName().equals("req")){
        parameterValues[i] = req;
    }else if(parameters[i].getName().equals("resp")){
        parameterValues[i] = resp;
    }else if(parameters[i].getName().equals("session")){
        parameterValues[i] = req.getSession();
    }else{
        String parameterName = parameters[i].getName();
        parameterValues[i] = req.getParameter(parameterName);
    }
}
```

但是这还没完。如果这个时候我们直接`method.invoke(controllerBeanObject,parameterValues)`的话，很容易出现下列异常：

![IllegalArgumentException](https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.4356l2iavo80.webp)

问题就出在，我们提供的参数类型和实际参数的类型对不上，`req.getParameter`得到的可以转成String,但是如果参数是Integer，那么就会报出上面的异常，处理方法就是**通过分析参数的类型信息，进行类型转换**，最后的代码如下：

```java
String parameterName = parameters[i].getName();
String parameterValue = req.getParameter(parameterName);
String typeName = parameters[i].getType().getName();
Object parameterObject = parameterValue;
if(parameterObject!=null){
    if("java.lang.Integer".equals(typeName)){
        parameterObject = Integer.parseInt(parameterValue);
    }else if(...){
        
    }
}
parameterValues[i] = parameterObject;
```

## 提取资源跳转

这部分就比较简单了，我们只要在Dispatcher中分析del的返回值就好了：

```java
Object returnObj = method.invoke(this,req,resp);
String methodReturnStr = (String) returnObj;

if(methodReturnStr.startsWith("redirect:")){
	String redirectStr = methodReturnStr.substring("redirect:".length());
	resp.sendRedirect(redirectStr);
}else{
    /*Thymeleaf模板跳转*/
	super.processTemplate(methodReturnStr,req,resp);
}
```

# 结语

以上的部分对实际开发并没有什么用，因为不可能比框架写得更好，考虑得更周到，但是其思路是值得借鉴和学习的，尤其是如何处理冗余的代码。
