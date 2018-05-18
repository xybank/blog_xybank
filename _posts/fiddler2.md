---
title: 抓包工具（Fiddler）的简单使用和实际的调试案例
date: 2018-05-17 23:10:23
tags: [dvlp123456,2018,抓包工具,技术]
categories: 余声赞
description: 抓包工具（Fiddler）的简单使用和实际的调试案例
---

## 来龙去脉介绍  

一直以来就想着了解下抓包这个东西了，一直知道这个对于开发和测试等是很有帮助的。经过网上资料的查询得知Fiddler这一款在Windows环境下比较好用，免费简单，故而决定尝试下这个软件。   
<!--more-->

## Fiddler软件的安装  

打开baidu搜索页面（www.baidu.com），搜索Fiddler。在结果页面中可以看到Fiddler 的官网，后面标记有“官网”字样的链接。点击进入官网，Fiddler官网首页的自我介绍中概括了它的显著特征：Telerik Fiddler The free web debugging proxy for any browser,system or platfor，很明显，这个软件也是可以调试安卓手机的。在官网进行free download，然后在本机安装即可。  

## Fiddler简单入门  

安装好Fiddler后，一开始不熟悉，所以可以在网上找一些简单入门的教学来进行简单入门。如，在baidu中搜索Fiddler入门等，我自己是在头条官网中搜索Fiddler简单入门，然后找一些简单入门的教学视频的，下面的一个教学视频是我的简单入门视频，地址：https://www.365yg.com/i6438548950357115393#mid=19159171029，当然了，如果是高手，请忽略这一步，直接使用。以上入门教学视频内容简单介绍：抓包工具对比；Fiddler界面介绍，抓手机请求设置等。下面资料是Fiddler中对于get和post请求的一些简单区分：https://www.cnblogs.com/yoyoketang/p/6719717.html。  

## Fiddler案例--一个简单案例的尝试  

经过上面的简单入门后，我们基本可以上手使用Fiddler了。一切以实际使用为目标，所以想着尝试下简单的抓请求和修改一下数据，故而在网上搜索一个简单例子，地址：https://blog.csdn.net/liuquan0071/article/details/51917893。这个例子是Fiddler 修改请求表单和响应数据，经过操作发现虽然会有一些不一样，但是大体一样的。经过这个案例，我们学会了简单的修改请求参数的修改和响应内容的修改。  

## Fiddler案例—抓手机的请求  

Fiddler默认是抓电脑上的请求的，如果要抓手机的请求，要做一些设置，设置步骤在上面的简单入门视频中有介绍，也可以自行网上搜索，如下面地址：https://jingyan.baidu.com/article/03b2f78c7b6bb05ea237aed2.html。  

Fiddler对Android应用进行抓包设置步骤简单介绍：  

### 1.Fiddler设置：如图  

![](/img/fiddler-1.png)  

### 2.手机设置：在已连上的WiFi中（这就要求电脑和手机连在同一个WiFi中了），设置代理，代理ip是电脑的ip，端口是Fiddler的端口（默认是8888）。  

经过以上步骤此时，操作手机的浏览器或者app会发现电脑上的Fiddler正在抓所有的请求了。以下图片展示了一个抓取请求和一些分析的实例：（特别声明：此资料仅仅用于学习，不可用于其他目的，否则造成一切后果自负）  

![](/img/fiddler-2.png)  
![](/img/fiddler-3.png)  
![](/img/fiddler-4.png)  
![](/img/fiddler-5.png)  
![](/img/fiddler-6.png)  

### 简单总结：  

Fiddler所有table tab和所有功能都可以去查看和尝试使用，这样就能更加切身的体会到每一个功能的用处了，如WebForms中可以看到中文等。  

## Fiddler软件使用到的一些快捷键和命令总结  

清除所有会话—ctrl+x；  
快速输入框获取焦点—ctrl+q；  
快速输入框清除所有会话—cls；  
快速输入框打断点—bpu，如bpu s.taobao.com，再次输入bpu（或者go）清除所有断点；  
