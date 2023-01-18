---
title: java多态与动态绑定
date: 2022-10-16 11:44:09
tags:
- java
- OOP
categories:
- java
banner_img: https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.tc1jjbxpfmo.webp
index_img: https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/image.tc1jjbxpfmo.webp
excerpt: 介绍多态机制以及java的实现
---

# 面向对象基础

我们先来聊聊什么是`面向对象（OOP）`。编程语言干的活绝大部分是<u>将实际的问题映射到机器模型之中</u>，而映射问题的思路是五花八门的。<u>面向对象</u>的基本思路就是**用编程语言去表达实际问题的元素，围绕这些元素进行操作**，这些元素在java中就被称为`对象`。

比如说我现在要用OOP的思想去描述一个苹果，那么我会说：这个苹果的ID=xxxxx，颜色=红色，种类=红富士，产地=某地，是否打过药=是，除此之外，我还会围绕这个苹果进行操作，比如把它卖给张三，又比如吃掉它，这就是一个**为实际问题的元素建立对象的过程**。像什么苹果的ID、颜色、产地等等都是这个对象的<u>状态（属性）</u>，而被卖掉或者是被吃掉就是这个对象的行为，每个对象是不同的，因此对象还有自己唯一的<u>标识</u>。

> **对象的定义**：一个对象具有自己的状态，行为和标识。这意味着对象有自己的内部数据(提供状态)、方法 (产生行为)，并彼此区分（每个对象在内存中都有唯一的地址）

一个苹果对象很好管理，那如果是成百上千的苹果对象呢？难道要一个个的给它们建立对象吗？这显然不现实，这就要提到`类(class)`的概念。一个地方产的苹果往往有许多特征是一样的，比如产地啊，种类啊，是否打过药啊，那么我们就可以将其**抽象**为一个类，比如`张三果园苹果`，**对象都从这个类生成，进行自己个性化的定制**。

我们使用<u>类</u>来将一个对象的各种内部数据和行为封装在一起，以此来表达实际问题的元素。但有一个问题没有解决：<u>如果我们现在有一批`杰克果园苹果`，是从外国进口的，这个时候再创建一个新类也太麻烦了吧？</u>

比较好的解决办法是什么呢？<u>我们不创建`张三果园苹果类`了，而是创建一个`苹果类`（**注意不是苹果对象**），其属性和行为和``张三果园苹果类`差不多，但是多个`果园名称`属性，然后张三果园和杰克果园产的苹果都<font color=purple>**继承**</font>自这个苹果类。</u>什么是继承呢，也就是**is a**的关系，杰克产的苹果是苹果，张三产的苹果也是苹果，他们都有苹果类的属性和行为，除此之外，还可以定制自己苹果的属性和行为。

以上的概念呢十分基础，但是还是要理理清的。

# 什么是多态

我们常常听到：<u>对象是类的实例化</u>，什么意思呢？打个比方，我们正在为一批刚摘下来的水果设定属性，我们操作的是真真正正的苹果，也就是苹果对象1，苹果对象2，苹果对象3，苹果对象4这样一个个数过来的，但是设定属性时，是根据苹果类一批次设定的，而不管它是苹果对象几。

拓展一下思路，实际上**我们处理对象，不仅不把它看成是其所属的类，还看成其所属的基类**。假如有一个水果类，其下面有苹果子类，香蕉子类，西瓜子类等等，<font color=purple>我们设定一个eat**方法**，我们就不需要知道其是什么水果类型，反正是水果都能吃</font>。这就是**多态**的思想，那么多态有什么好处呢？

我们来举个例子，如果Java不支持多态，水果们应该这些写：

```java
class Fruit{
    int id;
    String name;
    String color;
    String origin;
   	
    void setOrigin(String origin);
    void setColor(String color);
    
}

class Apple extends fruit{
    void eatApple();
}

class Watermelon extends fruit{
    void eatWatermelon();
}

class Banana extends banana{
    void eatBanana();
}
```

如果我们要吃个水果，每个水果类里都要定义一遍，如果调用这些方法，就会是下面这样：

```java
viod main(){
     Apple apple = new Apple();
    Watermelon watermelon = new Watermelon();
    //吃苹果
    apple.eatApple();
    //吃西瓜
   Watermelon.eatWatermelon();
}
```

如果使用多态的思想，编写的水果类会是下面这样：

```java
class Fruit{
    ...
    void eatFruit();
}

class Apple extends fruit{
    //不需要定义自己的方法
}

class Watermelon extends fruit{
   
}

class Banana extends banana{
    
}
```

现在吃水果的话：

```java
viod main(){
    Fruit apple = new Apple();
    Fruit watermelon = new Watermelon();
    //吃苹果
    apple.eatFruit();
    //吃西瓜
   Watermelon.eatFruit();
}
```

多态这层抽象使得软件系统更加好维护，可扩展性更强，加一个西瓜不需要我们再去加一个吃西瓜的方法，直接使用`eatFruit`方法就可以了。

# 重写(override)和重载(overload)

> - **重写(override)**：**子类对父类**方法的实现过程进行重新编写，但是**方法名，参数的类型和顺序是一样的**，和<u>多态</u>有关。
> - **重载(overload)**：**在一个类里面**，方法名相同，但是参数不同，比较常用的是构造器重载。

分清重写和重载的区别是很重要的，重写和本篇的主题多态有关，因为其涉及父子类相同的方法。举个例子，如果我们想定义自己吃苹果的方法，在java中就可以这么写：

```java
class Fruit{
    ...
    void eatFruit();
}

class Apple extends fruit{
    @override
    void eatFruit(){
        print("这是李四吃苹果的方法");
    }
}

class Watermelon extends fruit{

}

class Banana extends banana{
    
}
```

这就是多态中的override，这也引出一个核心问题：<font color=purple>计算机怎么知道我们要调用的是父类还是子类的方法呢？</font>这就要说到<u>绑定</u>的问题了。

# 多态在JAVA的实现

## 静态绑定和动态绑定

> **绑定**：将一个方法的调用和一个方法的主体关联起来，比如将eatFruit()方法和apple关联起来。

绑定分为**静态绑定**和**动态绑定**。

先来说说静态绑定，所谓的静态绑定<font color=purple>就是在**编译期**就进行方法和类的绑定</font>，一般用于<u>访问对象的属性</u>或者是<u>final、static和private修饰的方法</u>，这类方法的特征就是**确定无疑**。

动态绑定，就是<font color=purple>该对象的真实类型只有**运行时**才能确定，所以调用到的方法是运行时对象真实类型的方法</font>，在java中，除了静态绑定中提到的情况，其它都是动态绑定。

重写(override)就是动态绑定，也就是运行时绑定；重载(overload)就是静态绑定，也就是编译时绑定。

## 动态绑定实现多态

首先举一个经典例子,下面调用的是apple的方法还是基类Fruit的方法：

```java
viod main(){
    // apple实现了自己的eatFruit方法
    Fruit apple = new Apple();
    
    apple.eatFruit();
   	
}
```

因为子类apple定义了自己的eatFruit，所以调用的是apple的方法。由此可以看出，<u>当通过子类的实例去调用一系列的实例方法（包括一个方法调用的另一个方法）时，将优先和子类本身包含的实例方法动态绑定，如果子类没有定义这个实例方法，才会和从父类中继承来的实例方法动态绑定</u>。

JVM在准备阶段会生成一个<u>方法表</u>，里面存储着每个类的**每个方法对应的内存地址**，子类方法表中的方法包含父类的所有方法， 且**每个方法和父类方法的索引值（方法在方法表中的位置）相同**。比如父类Fruit的方法eatFruit的索引号是1，Apple子类如果没有重写该方法，那么该索引号1指向的就是父类Fruit的eatFruit；如果其重写了该方法，那么Apple子类方法表中的索引号1就指向自己的eatFruit。

在动态绑定的时，JVM会先根据动态类型获取对应类的方法表，再根据索引获取方法表中对应方法的内存地址。