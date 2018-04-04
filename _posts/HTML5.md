---
title: HTML5
date: 2018-04-04 13:54:07
tags: [hexo,HTML5,css3]
categories: 杨新才
description: HTML5语法参考
---
# 欢迎来学习HTML5
- **HTML5 表单**
- **HTML 5 应用程序缓存**
- **HTML 5 服务器发送事件**
<br/>
-
> HTML5 新的 Input 类型
HTML5 拥有多个新的表单输入类型。这些新特性提供了更好的输入控制和验证。
本章全面介绍这些新的输入类型。

- **email**
- **url**
- **number**
- **range**
- **Date pickers (date, month, week, time, datetime, datetime-local)**
- **search**
- **color**
<br/>
>HTML5 引入了应用程序缓存，这意味着 web 应用可进行缓存，并可在没有因特网连接时进行访问。
应用程序缓存为应用带来三个优势：

- **离线浏览 - 用户可在应用离线时使用它们**
- **速度 - 已缓存资源加载得更快**
- **减少服务器负载 - 浏览器将只从服务器下载更新过或更改过的资源。**
<br/>
>Server-Sent 事件指的是网页自动获取来自服务器的更新。以前也可能做到这一点，前提是网页不得不询问是否有可用的更新。通过服务器发送事件，更新能够自动到达。例子：Facebook/Twitter 更新、估价更新、新的博文、赛事结果等。

``` php 
var source=new EventSource("demo_sse.php");
source.onmessage=function(event)
  {
  document.getElementById("result").innerHTML+=event.data + "<br />";
  };
```
<br/>
-

#css3教程
>css3模块包括

- **选择器**
- **框模型**
- **背景和边框**
- **文本效果**
- **2D/3D 转换**
- **动画**
- **多列布局**
- **用户界面**
<br/>
##css3边框
>通过 CSS3，您能够创建圆角边框，向矩形添加阴影，使用图片来绘制边框 - 并且不需使用设计软件，比如 PhotoShop。在本章中，您将学到以下边框属性：

- **border-radius**
- **box-shadow**
- **border-image**
<br/>
