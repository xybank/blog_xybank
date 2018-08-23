---
title: 记录建站遇到的各种问题
date: 2018-08-20 09:54:00 
tags: [java] 
categories: 甘明阳
description: 各种问题，包括html乱码、数据库乱码、服务器速度优化、java流的换行
---

#### 中文乱码

##### 导入html文件乱码

用过sublime text3后我就迷上了sublime的快捷键，相比之下用myeclipse写感觉好慢，所以一般前端代码我都是用sublime写完后再导入到myeclipse里面，但是有的时候会有中文乱码的现象，由于myeclipse默认使用GBK的编码，我们需要改为

utf-8才能正常使用，有两处地方要改

1、改变servlet项目的编码为utf-8

右键项目，选择Priperties，搜索Resource，改变编码为utf-8

![Q图片2018062321051](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/java_pro_01.png)

2、改变myeclisp的编码为utf-8 ，找到window -> preference ->general ->workspace，将编码改为utf-8

![Q图片2018062321051](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/java_pro02.png)

这里也顺便解决了另外一个问题，就是js文件报错的问题

##### myeclipse打开html文件缓慢

由于myeclipse默认使用可视化打开，所以会消耗不少时间，我们需要改变打开方式

有两种方式

1、右键文件openwith ->MyEclipse HTML Editor

2、改变默认打开方式 

window -> preference -> 搜索file 找到File Associations,如图

![Q图片2018062321184](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/java_pro_03.png)

选中html文件，然后在下方框框里面选择MyEclipse HTML Editor,点击Default，就可以啦

![Q图片2018062321204](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/java_pro_04.png)

其他的文件觉得慢也可以按照这种思路做相应的改动

##### JDBC插入数据

##### 报错

查询的时候使用

```bash
stmt.executeQuery(sql);
```



update或者insert操作的时候，使用

```bash
stmt.execute(sql);
```

##### 数据库乱码

我之前插入数据的时候，数据的中文数据变成了？？？，在网上找了半天，发现网上很多答案有错误，这里指正一下

思路是在getConnection()里面增加两个设置编码的参数，useUnicode和characterEncoding

```bash
conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/users?useUnicode=true&characterEncoding=utf8");
```

但是网上很多答案都把characterEncoding=utf8写成了characterEncoding=utf-8，搞了我好久，这里特别强调一下。

#### 网站速度慢

在网站注册功能，登录功能和个人主页功能完成的时候，我发现了一个问题，网站速度慢，Google了一下，才发现我的请求全部用的重定向(头疼)，当初建站的时候没注意这一点(伤心)

这里说明一下请求重定向和请求转发的区别，比如说从a.servlet到b.servlet

重定向走的路： 浏览器->a.servlet->浏览器->b.servlet

转发走的路：浏览器 -> a.servlet -> b.servlet

重定向明显走的路多一点，这还只是一个流程，如果有多个流程，那速度简直是天差地别啦

下面总结一下重定向和转发的特点：

##### 请求转发：

请求转发的过程全部会在服务器完成，经过多个servlet流程浏览器也看不到，明显比较安全，而且每次转发都会把当前request和response域对象传过去，所有流程都是用的同一个request和response对象，数据好取，而且由于不用通知客户端，速度明显快很多，就是不能请求外部服务器。

##### 请求重定向：

可以请求外部服务器，由于我的站点不用请求别的服务器，这一点跟没有一样，可以说被请求转发全方面碾压。

浏览器上方url会改变，由于浏览器url改变我们的servlet不会暴露出来，对于服务器安全性有很大的提高，建议后台的最后一个请求使用重定向，前面所有请求用转发

##### 注意：

改了请求方式后在jsp要指定meta的href的值，不然会找不到样式，因为请求转发不会通知浏览器地址改变，也就是说浏览器用的地址一直都是第一个servlet的地址，下面是解决方法：

步骤一：在jsp上面加上这两句话

```java
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
```

步骤二： 在jsp的head标签里面要加`<base href="<%=basePath%>">`标签

request.getScheme()表示站点使用协议

request.getServerName()表示站点主机，本地就是localhost

request.getServerPort()表示站点端口号

request.getContextPath()表示站点项目名称

最后的basePath是一个完整的url，比如我的url是：

```bash
http://localhost:8080/servletPro/
```

然后将href指定为这个地址，以后写相对路径就是相对这个地址来的

##### java流换行问题

使用java流操作文件的时候，发现缓冲字符流BufferedWriter里面有一个方法readLine()可以一行一行的读取数据，相比FileWriter一个字符一个字符的读取快了很多，但是不会读取换行符号，需要我们手动添加换行符，由于在不同的操作系统上换行符不同：

```bash
mac:     \r     
linux:    \n
window:    \r\n
```

可以使用java里面的System.getProperty("line.separator")自动获取空格来解决这个问题



