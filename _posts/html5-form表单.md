---
title: html5-form表单
date: 2018-04-02 21:14:06
tags: [form表单，html5]
categories: 薛玉博
description: form表单的事件属性

---

### 1，前言

​          主要写一些我在学习form表单中遇到的问题和一些我自己的看法。

### 2，内容

```
<form onsubmit="return false">
    <input id="num_a" /> +
    <input id="num_b" /> =
    <output id="result" onforminput="resCalc()"></output>
</form>
```

​        这是我看到的一个例子，在这里我遇到了两个问题。

​       问题1：执行代码之后，无结果显示。

​        原因：onforminput部分浏览器不支持，不会执行resCalc方法，需要换另外一种写法。我换了下面一种写法：

```
<form onsubmit="resCalc()">
    <input id="num_a" /> +
    <input id="num_b" /> =
    <output id="result"></output>
    <br/>
    <input type="submit" />
</form>
```

​       这样我点击提交按钮就可以执行resCalc方法，但又引发了一个新的问题：页面  闪了一下变成空白页。我在网上查了一下原因，是因为onsubmit方法会自动刷新页面。所以执行方法之后跳到了空白页。我有改成另外一种方法终于可以了：

```
<form >
    <input id="num_a" /> +
    <input id="num_b" /> =
    <output id="result" ></output>
    <br/>
    <input type="button" value="提交" onclick="resCalc()" />
</form>

```

​       如果不想这样改，大家也可以在resCalc方法里做异步请求，来解决这个问题，这里就不多做描述。

问题2：onsubmit="return false"里的return false是什么意思。

 return false是在form表单只有一个输入框的时候，防止你在输入完成之后表单自动提交而增加的一个属性。

### 3，form表单事件属性

| 属性          | 值       | 描述                                  |
| ------------- | -------- | ------------------------------------- |
| onblur        | *script* | 当元素失去焦点时运行脚本              |
| onchange      | script   | 当元素改变时运行脚本                  |
| oncontextmenu | script   | 当触发上下文菜单时运行脚本            |
| onfocus       | script   | 当元素获得焦点时运行脚本              |
| onformchange  | script   | 当表单改变时运行脚本                  |
| onforminput   | script   | 当表单获得用户输入时运行脚本          |
| oninput       | script   | 当元素获得用户输入时运行脚本          |
| oninvalid     | script   | 当元素无效时运行脚本                  |
| onreset       | script   | 当表单重置时运行脚本。HTML 5 不支持。 |
| onselect      | script   | 当选取元素时运行脚本                  |
| onsubmit      | script   | 当提交表单时运行脚本                  |

### 4，网址

具体内容详见[W3school](http://www.w3school.com.cn)里的html和html5。