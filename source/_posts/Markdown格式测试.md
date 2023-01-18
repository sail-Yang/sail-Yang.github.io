---
title: Markdown样式测试
excerpt: 对fluid主题的markdown样式测试
date: 2022-09-23 08:39:10
tags: [Test]
categories: 
    - 其他
---

# 代码块

```java
public class Test{
    public static void main(String args[]){
        System.out.println("This a test.");
    }
}
```

# 标签

<p class="note note-primary">primary标签</p>

<p class="note note-info">info标签</p>

<p class="note note-success">success标签</p>

<p class="note note-danger">danger标签</p>

```markdown
语法1：<p class="note note-primary">primary标签</p>

语法2: 
{% note success %}
文字 或者 `markdown` 均可
{% endnote %}

行内标签： <span class="label label-primary">Label</span>
```

# 表格

```markdown
| 姓名  | 年龄  | 介绍  |
|:---:|:---:|:---:|
| a   | 20  | 1   |
| b   | 20  | 2   |
| c   | 18  | 3   |
```

| 姓名  | 年龄  | 介绍  |
|:---:|:---:|:---:|
| a   | 20  | 1   |
| b   | 20  | 2   |
| c   | 18  | 3   |

# 页面嵌套

```markdown
<iframe src="https://hexo.fluid-dev.com/" width="100%" height="500" name="topFrame" scrolling="yes"  noresize="noresize" frameborder="0" id="topFrame"></iframe>
```

<iframe src="https://hexo.fluid-dev.com/" width="100%" height="500" name="topFrame" scrolling="yes"  noresize="noresize" frameborder="0" id="topFrame"></iframe>

# 折叠标签

```markdown
<details>
<summary>这是一个折叠块</summary>
<p><b>行一</b></p>
<p><b>行二</b></p>
<img width="200" src="https://www.jianguoyun.com/c/dl-file/2022923102812.jpg?dt=rin5ma&sd=dlmq2&kv=MTE4NjU5OTA4MUBxcS5jb20&vr=1&ud=7ye0WsQXUj-HvwzVtVeuuWw0Lg28JNzUwHpN3F7Ivw4" alt="wechat">
</details>
```

<details>
<summary>这是一个折叠块</summary>
<p><b>行一</b></p>
<p><b>行二</b></p>
<img src="https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/凌波丽3.6folue9m3wk0.webp" alt="凌波丽3">
</details>

# 图片引用

![凌波丽3](https://cdn.staticaly.com/gh/sail-Yang/myImage@main/img/凌波丽3.6folue9m3wk0.webp)

# 脚注

```markdown
脚注演示[^1]
[^1]: 脚注内容演示
[^2]: 脚注内容演示

脚注演示[^2]
[^2]: 脚注内容演示
```

脚注演示[^1]
[^1]: 脚注内容演示
[^2]: 脚注内容演示

脚注演示[^2]
[^2]: 脚注内容演示
