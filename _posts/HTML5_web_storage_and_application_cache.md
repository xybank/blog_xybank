---
title: HTML5_web存储和应用缓存
date: 2018-04-17 13:14:15
tags: [HTML5]
categories: 匡利金
description: HTML5_web存储和应用缓存
---
## 一. 在客户端存储数据
### 1. HTML5 提供了两种在客户端存储数据的新方法：

**localStorage** - 没有时间限制的数据存储  
**sessionStorage** - 针对一个 session 的数据存储  
*之前，这些都是由 cookie 完成的。但是 cookie 不适合大量数据的存储，因为它们由每个对服务器的请求来传递，这使得 cookie 速度很慢而且效率也不高。* 

*在 HTML5 中，数据不是由每个服务器请求传递的，而是只有在请求时使用数据。它使在不影响网站性能的情况下存储大量数据成为可能。*  

*对于不同的网站，数据存储于不同的区域，并且一个网站只能访问其自身的数据。*

*HTML5 使用 JavaScript 来存储和访问数据*
### 2. localStorage 方法
**localStorage 方法存储的数据没有时间限制。第二天、第二周或下一年之后，数据依然可用。**

如何创建和访问 localStorage：
``` html5
<script type="text/javascript">
localStorage.lastname="Smith";
document.write(localStorage.lastname);
</script>
```
#### 案例：统计这个浏览器访问的次数  

``` html5
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>domo</title>
</head>
<body>

<div id="result"></div>
<script>
if(typeof(Storage)!=="undefined")
{
  localStorage.sitename="www.baidu.com";
  document.getElementById("result").innerHTML="网站名：" + localStorage.sitename;
}
else
{
  document.getElementById("result").innerHTML="对不起，您的浏览器不支持 web 存储。";
}
</script>

</body>
</html>
``` 
### 3. sessionStorage 方法
**sessionStorage 方法针对一个 session进行数据存储。当用户关闭浏览器窗口后，数据会被删除**。

如何创建并访问一个 sessionStorage：
``` html5
<script type="text/javascript">
sessionStorage.lastname="Smith";
document.write(sessionStorage.lastname);
</script>
``` 
#### *localStorage*和*sessionStorage*使用时使用相同的API：
1. **localStorage.setItem("key","value");//以“key”为名称存储一个值“value”**
2. **localStorage.getItem("key");//获取名称为“key”的值**
3. **localStorage.removeItem("key");//删除名称为“key”的信息。**

4. **localStorage.clear();//清空localStorage中所有信息**  

#### 案例：统计这个浏览器访问的次数
``` html5
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>demo</title>	
<script>
function clickCounter()
{
	if(typeof(Storage)!=="undefined")
	{
		if (localStorage.clickcount)
		{
			localStorage.clickcount=Number(localStorage.clickcount)+1;
		}
		else
		{
			localStorage.clickcount=1;
		}
		document.getElementById("result").innerHTML=" 你已经点击了按钮 " + localStorage.clickcount + " 次 !";
	}
	else
	{
		document.getElementById("result").innerHTML="对不起，您的浏览器不支持 web 存储。";
	}
}
</script>
</head>
<body>
<p><button onclick="clickCounter()" type="button">点我！</button></p>
<div id="result"></div>
<p>点击该按钮查看计数器的增加。</p>
<p>关闭浏览器选项卡(或窗口),重新打开此页面,计数器将继续计数(不是重置)。</p>
</body>
</html>
``` 

名称| localStorage| sessionStorage
---|---|----
相同点|使用一样的API|使用一样的API
不同点|除非数据被清除，否则一直存在| 页面会话期间可用  

参考资料：
[Web Storage API](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Storage_API)

