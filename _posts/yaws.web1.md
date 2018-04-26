---
title:  搭建基于yaws的erlang_web服务_1_环境搭建
date:  2018-04-26 12:51:25
tags: [yaws,erlang,web]
categories: 张永枫
description: 搭建基于yaws的erlang_web服务
---

# 搭建基于yaws的erlang web服务
## yaws的安装

---
### windows下安装

yaws window的下载地址[http://yaws.hyber.org/download/windows/](http://yaws.hyber.org/download/windows/)
直接下载后双击.exe文件安装

![](/img/zyf_img/Snip20180426_1.png)

点击“next”按钮

![](/img/zyf_img/Snip20180426_2.png)

选择“I accept the agreement”，点击“next”

![](/img/zyf_img/Snip20180426_3.png)

选择安装路径同，点击“next”，进入下一步

![](/img/zyf_img/Snip20180426_4.png)

点击“next”,进入下一步

![](/img/zyf_img/Snip20180426_5.png)

点击“finish” 完成安装

![](/img/zyf_img/Snip20180426_6.png)

### mac安装

yaws mac 直接用homebrew 安装即可

```
brew install yaws
```

## 启动yaws服务

---

在mac系统示例如下：
进入命令窗口，输入
```
yaws -i 
```

会出现以下启动信息

```
➜  blog yaws -i 
Erlang R15B03 (erts-5.9.3.1) [source] [64-bit] [smp:8:8] [async-threads:0] [hipe] [kernel-poll:true]

Eshell V5.9.3.1  (abort with ^G)
1> 
=INFO REPORT==== 26-Apr-2018::10:47:33 ===
Yaws: Using config file /usr/local/etc/yaws/yaws.conf

=INFO REPORT==== 26-Apr-2018::10:47:33 ===
yaws debug:Add path "/usr/local/var/yaws/ebin"

=INFO REPORT==== 26-Apr-2018::10:47:33 ===
yaws debug:Add path "/usr/local/lib/yaws/examples/ebin"

=INFO REPORT==== 26-Apr-2018::10:47:33 ===
yaws debug:Add path "/usr/local/lib/yaws/examples/ebin"

=INFO REPORT==== 26-Apr-2018::10:47:33 ===
yaws debug:Running with id="default" (localinstall=false) 
Running with debug checks turned on (slower server) 
Logging to directory "/usr/local/var/log/yaws"

=INFO REPORT==== 26-Apr-2018::10:47:33 ===
Ctlfile : /Users/Macx/.yaws/yaws/default/CTL

=INFO REPORT==== 26-Apr-2018::10:47:33 ===
Yaws: Listening to 0.0.0.0:8089 for <2> virtual servers:
 - http://feng.local:8089 under /usr/local/var/yaws/www
 - http://localhost:8089 under /tmp

=INFO REPORT==== 26-Apr-2018::10:47:33 ===
Yaws: Listening to 0.0.0.0:4002 for <1> virtual servers:
 - https://feng.local:4002 under /tmp
```

其中yaws默认的配置文件如下：
/usr/local/etc/yaws/yaws.conf

yaws服务启动的端口为8089，（默认为8080）
访问http://feng.local:8089可以得到如下页面，web服务启动成功
![](/img/zyf_img/Snip20180426_9.png)

## 编写第一个页面
---
进入/usr/local/var/yaws/www
新建一个first.yaws文件
输入内容如下：

```
<!DOCTYPE html>  
<html>  
<head>  
    <title>my first page</title>  
</head>  
<body>  
    <erl>  
        out(Arg) ->  
            {html,"<p>my first page！</p>"}.  
    </erl>  
</body>  
</html>  
```

![](/img/zyf_img/Snip20180426_11.png)

## 用erlang代码实现动态页面
---
<erl></erl>嵌入Erlang代码来定义函数，然后使用out(Args)函数来调用它，以达到实现动态页面目的

可以在/usr/local/var/yaws/www/下新建my_out.erl
输入以下内容：

```
 -module(my_out).  
 -export([out/0]).  
   
 out() ->  
     "<p>Hello Out!</p>".  
```
编译erl文件my_out.erls
进入到yaws启动的命令窗口后执行

```
c(my_out).
```
访问 http://feng.local:8089/first.yaws 可以得到
![](/img/zyf_img/Snip20180426_12.png)






