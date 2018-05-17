---
title: JS-timer
date: 2018-05-17 15:12:04
tags: [js，定时器]
categories: 薛玉博
description: js定时器

---

### 1，js定时器

js 定时器有以下两个方法：

- setInterval() ：按照指定的周期（以毫秒计）来调用函数或计算表达式。方法会不停地调用函数，直到 clearInterval() 被调用或窗口被关闭。
- setTimeout() ：在指定的毫秒数后调用函数或计算表达式。

### 2，setInterval（）

```
setInterval(code,millisec,lang)
```

| 参数     | 描述                                                    |
| :------- | ------------------------------------------------------- |
| code     | 必需。要调用的函数或要执行的代码串。                    |
| millisec | 必须。周期性执行或调用 code 之间的时间间隔，以毫秒计。  |
| lang     | 可选。脚本语言可以是：JScript ， VBScript ， JavaScript |

代码：

```
<html>
<body>
<input type="text" id="clock" />
<script type="text/javascript">
var int=self.setInterval("clock()",1000);
function clock()
{
var d=new Date();
var t=d.toLocaleTimeString();
document.getElementById("clock").value=t;
}
</script>
<button onclick="int=window.clearInterval(int)">停止</button>
</body>
</html>
```

### 3，setTimeout()

```
setTimeout(code,millisec,lang)
```

| 参数     | 描述                                                   |
| -------- | ------------------------------------------------------ |
| code     | 必需。要调用的函数后要执行的 JavaScript 代码串         |
| millisec | 必需。在执行代码前需等待的毫秒数。                     |
| lang     | 可选。脚本语言可以是：JScript ， VBScript ，JavaScript |

代码：

```
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>
<p>点击按钮，在等待 3 秒后弹出 "Hello"。</p>
<button onclick="myFunction()">点我</button>
<script>
function myFunction()
{
    setTimeout(function(){alert("Hello")},3000);
}
</script>
</body>
</html>
```

