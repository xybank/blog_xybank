---
title:  Navicate for Mysql的破解
date: 2018-04-15 20:05:00 
tags: [mysql] 
categories: 甘明阳
description: navcate从安装到使用一套讲解 
---

开发项目的时候经常会查询数据库的状态，每次都跑云服务器上查询感觉很麻烦，我现在郑重介绍一个好用的工具navicate，用过这个工具我真是爱上它啦，“这人用mysql不搭配navicate是不是傻”，工作中我经常会有这种想法，除了建表的便捷性，还可以设置各种查询彻底解放你的双手哦~

####  下载安装

首先这个工具需要我们的电脑上安装了mysql,如果你的电脑上没有安装mysql，点这里下载免安装版的：[点击下载](https://pan.baidu.com/s/1cteUV_MZ-rJGhg8x4S0eRA)

密码是：o2mb

在D盘创建一个文件夹mysql,然后解压到这个文件夹，双击bin下面的mysqld.exe打开，如图

![Q图片2018041520390](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql01.png?raw=true)

会有一个黑窗口一闪而过，我们不用管他，打开任务管理器看看是不是有一个mysqld.exe的进程在运行，如图：

![Q图片2018041520420](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql02.png?raw=true)

如果有就说明你的mysql已经成功运行，接下来我们下载Navicate for Mysql和相关的破解文件，[点击下载](https://pan.baidu.com/s/19fh2z8tkI-FCiL_XNKJqsw)

密码：5t6d

下载找到navicat111_mysql_cs_x86.exe，双击安装，安装后找到安装的路径，会多一个文件夹Navicat for MySQL，然后我们把压缩包里面的PatchNavicat.exe复制到Navicat for MySQL下，如图

![Q图片2018041520531](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql03.png?raw=true)

#### 开始使用

双击打开，选中navicte.exe，弹出一个success的框框就说明破解成功，然后我们双击navicte.exe打开，点击连接，如图：

![Q图片2018041520584](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql04.png?raw=true)

选择mysql,输入如图，密码是root

![Q图片2018041521003](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql05.png?raw=true)

#### 建表

首先建立数据库users：

![indow_mysql0](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql05.png?raw=true)

建表，首先设置id为主键，点钥匙就可以取消主键

![Q图片2018062021524](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql07.png?raw=true)



点击添加栏位添加新列

![Q图片2018062021540](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql08.png?raw=true)

保存，写表的名字，注意公司都是以T_ 开头，复数结尾，比如T_students，前面不能有空格

![Q图片2018062022050](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql10.png?raw=true)

#### 修改表

注意这里不能双击表名，要右键选择设计表，假如我想删除一列，右键这里点删除栏位

![Q图片2018062022093](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql11.png?raw=true)

如果像查看表里面有哪些数据，直接双击表名就可以了

![Q图片2018062022111](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql12.png?raw=true)



现在啥都没有，我们插入两条数据，点击加好可以添加一行新的

![Q图片2018062022131](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql13.png?raw=true)

基本操作就这些，然后是关于mysql的存储引擎，最常用的是InnoDB和MyISAM，MyISAM效率较高，但是不支持事务，外键约束等属性，因此一般都用InnoDB，新版本默认使用InnoDB，建表的时候想要选择引擎点击选项，如图

![Q图片2018062022210](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql14.png?raw=true)

建完表后想要查看引擎，对表右键选择对象信息

![Q图片2018062022225](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql15.png?raw=true)

#### 命令行使用

切换到查询

![Q图片2018062121475](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql16.png?raw=true)

点击新建查询

![Q图片2018062121494](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql17.png?raw=true)

##### 查询

![Q图片2018062121552](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql18.png?raw=true)

##### 增加数据

![Q图片2018062121572](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql19.png?raw=true)

注意字符串要用单引号引起来，修改也可以

![Q图片2018062122105](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql20.png?raw=true)

##### 快捷查询

点击保存，你的sql语句就保存下来了

![Q图片2018062810152](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql22.png?raw=true)



以后想要查询所有数据，点击查询->查询所有数据->运行就可以啦

![Q图片2018062810180](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/window_mysql21.png?raw=true)