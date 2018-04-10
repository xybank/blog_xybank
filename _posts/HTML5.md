---
title: HTML5入门
date: 2018-03-28 14:10:23
tags: [Nodejs]
categories: 杨新才
description: HTML5简单入门
---

## 1. HTML5 中的新特性  
 + 用于绘画的 canvas 元素  
 + 用于媒介回放的 video 和 audio 元素  
 + 对本地离线存储的更好的支持  
 + 新的特殊内容元素，比如 article、footer、header、nav、section  
 + 新的表单控件，比如 calendar、date、time、email、url、search  

`注意：此文档是本人看视频学习的手记。`    

## 2.HTML5 新的 Input 类型  

### （1）[官网：http://www.w3school.com.cn](http://www.w3school.com.cn)。  

HTML5 拥有多个新的表单输入类型。这些新特性提供了更好的输入控制和验证。
本章全面介绍这些新的输入类型：  

 + email  
 + url  
 + number  
 + range  
 + Date pickers (date, month, week, time, datetime, datetime-local)  
 + search  
 + color  
### （2）HTML5 的新的表单元素：  
HTML5 拥有若干涉及表单的元素和属性。本章介绍以下新的表单元素：

 + datalist  
datalist 元素规定输入域的选项列表。列表是通过 datalist 内的 option 元素创建的。如需把 datalist 绑定到输入域，请用输入域的 list 属性引用 datalist 的 id：

 + keygen  
keygen 元素的作用是提供一种验证用户的可靠方法。keygen 元素是密钥对生成器（key-pair generator）。当提交表单时，会生成两个键，一个是私钥，一个公钥。私钥（private key）存储于客户端，公钥（public key）则被发送到服务器。公钥可用于之后验证用户的客户端证书（client certificate）。目前，浏览器对此元素的糟糕的支持度不足以使其成为一种有用的安全标准。

 + output  
output 元素用于不同类型的输出，比如计算或脚本输出：

### （3）HTML5 的新的表单属性    
 
 + 新的 form 属性：   
1.autocomplete  
2.autocomplete  

 + 新的 input 属性：  
1.autocomplete  
2.autofocus  
3.form  
4.form overrides (formaction, formenctype, formmethod, formnovalidate, formtarget)  
5.height 和 width  
6.list  
7.min, max 和 step  
8.multiple  
9.pattern (regexp)  
10.placeholder  
11.required  

**在客户端存储数据**  

 + HTML5 提供了两种在客户端存储数据的新方法：  
1.localStorage - 没有时间限制的数据存储  
2.sessionStorage - 针对一个 session 的数据存储之前，这些都是由 cookie 完成的。但是 cookie 不适合大量数据的存储，因为它们由每个对服务器的请求来传递，这使得 cookie 速度很慢而且效率也不高。在 HTML5 中，数据不是由每个服务器请求传递的，而是只有在请求时使用数据。它使在不影响网站性能的情况下存储大量数据成为可能。对于不同的网站，数据存储于不同的区域，并且一个网站只能访问其自身的数据。HTML5 使用 JavaScript 来存储和访问数据。  

**localStorage 方法**  
localStorage 方法存储的数据没有时间限制。第二天、第二周或下一年之后，数据依然可用。如何创建和访问 localStorage：  

## sessionStorage 方法  

 +sessionStorage 方法针对一个 session 进行数据存储。当用户关闭浏览器窗口后，数据会被删除。如何创建并访问一个 sessionStorage：  

## 3.HTML 5 应用程序缓存

HTML5 引入了应用程序缓存，这意味着 web 应用可进行缓存，并可在没有因特网连接时进行访问。应用程序缓存为应用带来三个优势：	  

 + 离线浏览 - 用户可在应用离线时使用它们  
 + 速度 - 已缓存资源加载得更快  
 + 减少服务器负载 - 浏览器将只从服务器下载更新过或更改过的资源。  

## 4. HTML 5 服务器发送事件  

### 1.Server-Sent 事件 - 单向消息传递  
erver-Sent 事件指的是网页自动获取来自服务器的更新。以前也可能做到这一点，前提是网页不得不询问是否有可用的更新。通过服务器发送事件，更新能够自动到达。  

### 2.接收 Server-Sent 事件通知  

 + 创建一个新的 EventSource 对象，然后规定发送更新的页面的 URL（本例中是 "demo_sse.php"）  
 + 每接收到一次更新，就会发生 onmessage 事件  
 + 当 onmessage 事件发生时，把已接收的数据推入 id 为 "result" 的元素中

## 5. HTML5 提供了播放音频的标准。  

直到现在，仍然不存在一项旨在网页上播放音频的标准。今天，大多数音频是通过插件（比如 Flash）来播放的。然而，并非所有浏览器都拥有同样的插件。HTML5 规定了一种通过 audio 元素来包含音频的标准方法。audio 元素能够播放声音文件或者音频流。
---
title: HTML5
date: 2018-04-10 10:40:31
tags:
---
