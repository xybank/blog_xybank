---
title: html简述及html5新特性
date: 2018-06-04 10:23:43
tags: [HTML5]
categories: 甘潇敏
description: 简单记录html及html5新特性
photos: 
     "http://img.zcool.cn/community/0142135541fe180000019ae9b8cf86.jpg@1280w_1l_2o_100sh.png"
---

## HTML标准结构

html：超文本标记语言，以.html或者.htm为后缀，由浏览器解析执行

### 文档类型声明

指定html的版本和风格

```html
<!doctype html>
```
### html页面

#### 组成

由一对根标记组成，在根标记中包括两部分：文件头定义网页全局信息，文件主体显示页面的主要内容

```html
<!DOCTYPE html>				<!--文档类型声明-->
<html>					    <!--根标记-->
	<head></head>			<!--文件头-->
	<body></body>			<!--文件主体-->
</html>
```

#### 特殊符号

```html
 &nbsp;   			<!-- 空格 -->
 &lt;     			<!-- <	 -->
 &gt;     			<!-- >	 -->
 &copy;   			<!-- ?	 -->
 &yen;    			<!-- ￥	 -->
```

#### 文本

##### 	文本标记	

```html
<i></i>      		<!-- 倾斜 -->
<u></u>       		<!-- 下划线 -->
<s></s>      		<!-- 删除线 -->
<b></b>      		<!-- 加粗 -->
<sup></sup>  		<!-- 上标  -->
<sub></sub>  		<!-- 下标 -->
```

##### 	标题元素

```html
<hn></hn>      <!-- n:1-6数字 -->
<h1></h1>      <!-- 一级标题 -->
```

##### 	段落标记（块元素）

```html
<p>内容</p>
```

以段落的形式显示文本，文字大小采用默认大小，段落元素独占成行，而且距离其他元素会有垂直空白间距

##### 	换行元素

```html
<br>
```

##### 	分割线元素

```html
<hr>
```

##### 	预格式化

保留源文件中的格式，保留源文件的换行和空格效果

```html
<pre></pre>
```

##### 	分区元素

​	（1）块分区，div，形式：1.独立成行；2.无任何文本显示效果   作用：布局

​	（2）行分区，span，形式：1.多个span元素在一行中显示； 2.无任何特殊效果  作用：处理一行文字的不同效果

​	（3）行内元素与块级元素

​			1.行内元素：多个元素在一行内显示；ex:span,i,b,u,s,sup,sub,a,img

​			2.块级元素：每个块级元素会独自成行，即前后有换行的效果；ex:div,p,h1-h6,（结构标记）  作用：为了布局

#### 图像

##### 	图像格式

​	   jpg（jpeg）: 压缩；png : 背景透明；gif : 动画；

##### 	图像元素

​		1.语法：img

​		2.属性： src:要显示图片的路径（url）; width:宽度；height:高度；title:鼠标移入到元素上时，提示文字；alt:图片出错时显示文字

#### input

```html
<input type="text">			<!-- 文本框 -->
<input type="password">			<!-- 密码框 -->
<input type="radio">			<!-- 单选框 -->
<input type="checkbox">			<!-- 复选框 -->
<input type="submit">			<!-- 提交按钮 -->
<input type="reset">			<!-- 重置按钮 -->
<input type="button">			<!-- 普通按钮 -->
<input type="image">			<!-- 图片按钮 -->
<input type="hidden">			<!-- 隐藏域（提交给服务器，用户不看的数据） -->
<input type="file">			<!-- 文件选择框（用于选择文件上传） -->
```

## html5新特性

(1)新的语义标签

(2)增强型表单

(3)视频、音频

(4)Canvas绘图  （位图/矢量图）

(5)SVG绘图

(6)地理定位

(7)拖放API

(8)Web Woker

(9)Web Storage

(10)Web Socket

### 1.结构标记（语义标签）

​    在HTML5中，专门提出的一组来表示网页的结构。

​    和div是同样的作用，优点是提升布局代码的语义性和可读性，看到结构标记就可以知道这些标记是什么作用，用在哪里；缺点是不如div方便，div是所有的地方都可以用，约定俗成，而结构标记局限性较大，在功能上没有什么不同。

```html
<header></header>		 	 <!-- 页眉 -->
<nav></nav>			     	 <!-- 导航 -->
<aside></aside>			 	 <!-- 边栏 -->
<footer></footer>		 	 <!-- 底部信息 -->
<section></section>		  	 <!-- 节，主体内容 -->
<article></article>		  	 <!-- 简短的内容，如：论坛帖子 -->
```

### 2.增强表单

#### 1.input新增type类型

```html
<input type="email">			<!-- 电子邮件 -->
<input type="search">			<!-- 搜索框 -->
<input type="url">			<!-- url类型，数据必须符合url规范 -->
<input type="tel">			<!-- 电话号码 -->
<input type="number">			<!-- 数字类型 -->
<input type="range">			<!-- 范围类型（允许选择指定范围内的一个值） -->
<input type="color">			<!-- 颜色类型（颜色拾取控件） -->
<input type="date">			<!-- 日期类型（允许用户选择日期） -->
<input type="week">			<!-- 周类型（允许用户选择周） -->
<input type="month">			<!-- 月份类型（允许用户选择月份） -->
```

#### 2.新表单元素

​	H4中表单元素:input/textarea/select/option/label
   	H5新表单元素:datalist/progress/meter/output

##### (1)、datalist

```html
<datalist id="list1">
    <option value="rou">京酱肉丝</option>
    <option value="rou">鱼香肉丝</option>
    <option value="fish">水煮鱼</option>
    <option value="tudou">小炒土豆丝</option>
</datalist>
请输入您需要的午餐
<input type="text" list="list1" name="lunch"/>
<input type="submit" value="提交">
```

##### (2)、progress(进度条)

```html
<form action="#">
   网络连接中：<progress></progress><br/>		<!-- 左右晃动的进度条 -->
   下载进度值：<progress value="0.7"></progress><br/>	 <!--有指定进度值的进度条-->
</form>
```

##### (3)、meter（度量衡，刻度尺）

用于标示一个值所处范围：不可收受(红色)，可以接受(黄色)， 非常优秀(绿色)。

```html
<meter min="可取最小值" max="可取最大值" low="合理的下限值" high="合理上限值"   optimum="最佳值" value="当前实际值"></meter>
```

```html
<form action="#">
    机油含量：
    <meter id="m1" min="0" max="100" low="30" high="70" optimum="60" value="10"></meter>
</form>
```

##### (4)、output（输出）

语义标签，没有任何外观样式，样式上等同span，功能上等同于span，语义标签的优缺点前文已经提过，在此不再赘述。

#### 3.表单元素新属性

H4的属性有 id/class/title/style/type/value/name/readonly/disabled/checked

H5中表单元素新属性

##### placeholder：占位字符，在文本框未输入时起提示作用

```html
<input value="tom" placeholder="请输入用户名" />
```

##### autofocus：自动获取输入焦点

```html
<input autofocus />
```

##### multiple: 允许输入框中出现多个输入(如有多个输入，用逗号分隔)

```html
<input type="email" multiple/>
```

##### form:为一个元素指定form属性，值为某个表单的ID，则此输入域可以放到表单的外部

```html
<form id="f5"></form>
<input form="f5"/>
```

##### autocomplete: on/off ,自动补全，是否自动记录之前提交的数据，以用于下一次输入建议 

```html
<input autocomplete="on"/>		<!-- 自动补全 -->
```

##### required：false/true，必需的/必填项，在表单提交时会验证是否有输入，没有输入则弹出提示消息

```html
 <input required>		<!-- 必填项 -->
```

##### maxlength：最大长度，在有输入的情况下，限定输入域中字符的个数 

##### minlength：最小长度，在有输入的情况下，限定输入域中字符的个数，不是HTML5标准属性，仅部分浏览器支持（如Chrome） 

```html
<input maxlength="9" minlength="6">		<!-- 最少长度6，最大长度9 -->
```

##### min：限定输入的数字的最小值 

##### max：限定输入的数字的最大值

##### step：限定输入的数字的步长，与min属性连用 

```html
<input min="3"  max="42" step="2"> 		<!-- 最小值为3，最大值为42，步长为2 -->	
```

##### pattern：指定一个正则表达式，对输入进行验证 

```html
1. <input type="text" pattern="1[8475]\d{9}"> 
```

### 3.视频和音频

#### 视频 video：默认300*150的inline-block元素

```html
<video src="res/birds.mp4"></video>
<video>
      <source src="res/birds.mp4" />
      <source src="res/birds.ogg" />
      <source src="res/birds.webm" />
      您的浏览器不支持VIDEO播放!
</video>
```

autoplay:false  不自动播放

controls:false  不显示播放控件

loop:false     不循环播放

muted:false   不是静音播放

poster:" "    在播放第一帧之前显示的海报

preload:      视频预加载策略，可取值

auto:预加载视频的元数据以及缓存一定时长

metadata:仅加载视频元数据

none:不预加载任何数据

#### 音频播放 audio：默认300*30的inline-block元素

```html 
<audio src="res/bg.mp3"></audio>
<audio>
    <source src="res/bg.mp3" />
    <source src="res/bg.wav" />
</audio>
```

autoplay:false  是否自动播放 

controls:false   是否显示播放控件

loop:false      是否循环播放

muted:false    是否是静音

preload:       音频预加载策略

auto:       预加载音频元数据以及缓存一定时长

metadate:   仅预加载音频元数据

none:       不预加载作何数据

### 4.Canvas绘图技术

#### 网页中可用的绘图技术

(1)、SVG绘图:2D矢图绘图技术2000年出现，后纳入H5标准

(2)、Canvas绘图:2D位图绘图技术，H5提出绘图技术

(3)、WebGL绘图:3D绘图技术，尚未纳入H5标准

Canvas：画布，浏览器中默认300*150的inline-block元素，画布宽度和高度只能使用HTML/JS属性来赋值，不能使用CSS样式赋值!

#### 使用canvas绘制图形

```js
var ctx = canvas.getContext("2d");  //得到画布上的画笔对象
ctx.lineWidth = 1;       		   //描边宽度(线宽)
ctx.fillStyle = "#ff0";    		   //填充样式/颜色
ctx.strokeStyle = "#000"   		   //描边样式
ctx.fillRect(x,y,w,h);      	   //填充一个矩形
ctx.strokeRect(x,y,w,h);   		   //描边矩形
ctx.clearRect(x,y,w,h);    		   //清除一个矩形范围所有绘图
```

#### 使用canvas绘制一段文本

```js
ctx.textBaseline = "alphabetic";  //文本基线
ctx.font = "12px sans-serif";     //文本大小和字体
ctx.fillText(str,x,y);            //填充一段文本
ctx.strokeText(str,x,y);          //描边一段文本
ctx.measureText(txt);             //基于当前文字大小字体测量文本，返回一对象 {width:x}
```

#### canvas绘图中使用渐变对象

```html
<canvas id="cv" width="500" height="400">
    您的浏览器不支持canvas！
</canvas>
<script>
//    1.获取画笔
    var ctx=cv.getContext("2d");
//    2.创建渐变对象
    var g=ctx.createLinearGradient(0,0,500,400);
//    3.添加颜色点
    g.addColorStop(0,"#f00");
    g.addColorStop(0.5,"#ff0");
    g.addColorStop(1,"#0f0");
//    4.将渐变对象设置填充样式
    ctx.fillStyle=g;
//    5.创建矩形
    ctx.fillRect(0,0,500,400);
</script>
```

#### 使用canvas进行绘图--路径

```html 
<canvas id="cv" width="500" height="400">
    您的浏览器不支持canvas！
</canvas>
<script>
    //1.创建画笔
    var ctx=cv.getContext("2d");
    //x轴
    ctx.beginPath();
    ctx.strokeStyle="lightBlue";
    ctx.lineWidth=3;
    ctx.moveTo(50,350);
    ctx.lineTo(50,100);
    ctx.lineTo(45,105);
    ctx.lineTo(50,100);
    ctx.lineTo(55,105);
    ctx.stroke();
    //y轴
    ctx.beginPath();
    ctx.strokeStyle="lightGreen";
    ctx.moveTo(50,350);
    ctx.lineTo(350,350);
    ctx.lineTo(345,345);
    ctx.lineTo(350,350);
    ctx.lineTo(345,355);
    ctx.stroke();
</script>
```

#### 使用canvas进行绘图--图片

```html
<canvas id="cv" width="500" height="400">
    您的浏览器不支持canvas！
</canvas>
<script>
//1.获取画笔
    var ctx=cv.getContext("2d");
//    2.创建图片对象
    var p3=new Image();
//    3.对图片对象属性src赋值
    p3.src="images/p3.png";
    console.log("1:"+p3.width);
//    4.绑定事件onload 图片加载完成
    p3.onload=function(){
//    5.绘制图片
        console.log("2:"+p3.width);
        ctx.drawImage(p3,0,0);
        ctx.drawImage(p3,300,0);//右上角
        ctx.drawImage(p3,0,300);
        ctx.drawImage(p3,300,300);
        ctx.drawImage(p3,150,150);//中央
    }
</script>
```



**此篇博客只记录前4个html5新特性，后续请等待。**