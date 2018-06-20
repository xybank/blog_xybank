---
title: 云服务器ECS建站攻略
date: 2018-06-20 21:54:00 
tags: [java] 
categories: 甘明阳
description: 基于java环境和tomcat搭建服务器
---

最近买了阿里云的ECS产品，之前一直没有建站，因为我以为要先备案才能访问服务器，浪费了大半个月的时间(捂脸)。

这里先纠正一下概念，不管有没有备案，买了服务器就可以搭建自己的网站，只是地址不好记住，比如说我的地址是

43.25.118.37，那我的网站地址可能是

```bash
http://43.25.118.37:8080
```

如果我购买了一个域名ganmyds.cn，成功备案并且解析到43.25.118.37上,那我的网站地址就是

```bash
http://ganmyds.cn:8080
```

这样相对来说比较好记住。

由于我是学java的，自然建站后台也是选择java环境，这里强调一下我的环境

linux系统centos6.8 64位+jdk1.8.0+tomcat8.0

其他环境这里就不说了，下面开始环境搭建流程：

#### ECS管理软件下载

这里推荐两款软件用来管理ECS服务器，使用xshell远程连接终端，用xftp可以从你的电脑传文件到云服务器上

下载： [xshell](https://pan.baidu.com/s/1yyZljAoWGlY4iWHyzNXgzA) [xftp](https://pan.baidu.com/s/1JoRdHywfsIJQL93_vuVU9Q)

#### 软件使用方法

双击xshell.exe，新建会话

![Q图片2018060614551](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/aly_tomcat_01.png)

![Q图片2018060614573](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/aly_tomcat_02.png)

![Q图片2018060615001](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/aly_tomcat_03.png)

帐号密码没有输错就可以进入，然后我们要传文件的话，点击传输新文件就可以打开xftp

![Q图片2018060615012](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/aly_tomcat_04.png)

![Q图片2018060615031](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/aly_tomcat_05.png)

这里xftp的地址和你的linux地址是一致的，假如你想传文件到/usr/java/tomcat/里面，可以在xshell里面先切换到这里去

```bash
cd /usr/java/tomcat/
```

然后在点击传输新文件就可以快速传输文件了

#### 环境搭建

由于tomcat是用java写的，所有我们肯定要先搭建java环境，[下载jdk](https://pan.baidu.com/s/1PnLFVj05uy5PMw7cOOUsoQ) [下载tomcat](https://pan.baidu.com/s/1NnmsNf_4wOXMguFOO9Bimg) 

##### 创建目录

```bash
cd /usr     
mkdir java
cd java
mkdir tomcat
mkdir jdk
```

然后我们把jdk-8u171-linux-x64.tar.gz.tar.gz.tar.gz放到jdk里面，把apache-tomcat-8.5.31.tar.gz放到tomcat里面

##### 配置jdk环境变量

```bash
cd jdk
tar -zxvf jdk-8u171-linux-x64.tar.gz.tar.gz.tar.gz
vi /etc/profile
```

在里面对应路径加上下面内容

```bash
export JAVA_HOME=/usr/java/jdk/jdk1.8.0_171
export JRE_HOME=/usr/java/jdk/jdk1.8.0_171/jre
export CLASSPATH=.:$JAVA_HOME/lib$:JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin/$JAVA_HOME:$PATH
```

如图：

![Q图片2018060615260](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/aly_tomcat_06.png)

退出登录

```bash
logout
```

关闭xshell重新打开就可以了

```bash
java -version
```

如图就表示成功了

![Q图片2018060615300](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/aly_tomcat_07.png)

##### 安装tomcat

```bash
cd /usr/java/tomcat/
tar -zxvf apache-tomcat-8.5.31.tar.gz
cd apache-tomcat-8.5.31.tar.gz/bin
vi setclasspath.sh
```

在最下面加上内容

```bash
export JAVA_HOME=/usr/java/jdk/jdk1.8.0_171
export JRE_HOME=/usr/java/jdk/jdk1.8.0_171/jre
```

启动tomcat

```bash
./startup.sh
```

有Tomcat started就表示成功启动

![Q图片2018060616252](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/aly_tomcat_08.png)

然后在浏览器输入http://云服务器的ip:8080就可以访问了

好的，服务器终于搭建好了，由于tomcat默认是8080端口，我们的网站需要输入端口号，这个对于网站的推广是不好的，我们现在改为80端口

#### 修改配置文件

在window上只需要编辑tomcat/conf/server.xml，找到

```bash
<Connector URIEncoding="UTF-8" port="8080" protocol="HTTP/1.1"  
           connectionTimeout="20000"  
           redirectPort="8443" /> 
```

把port改为80就可以了

假如你的用户不是root这样不行，因为linux上非root用户不能监听1024以下的端口号，我们可以用linux的端口转发机制，输入

```bash
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080 
```

保存

```bash
service iptables save 
```

#### 安全组改动

如图修改

![Q图片2018060522013](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/aly_tomcat_09.png)

![Q图片2018060522033](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/aly_tomcat_10.png)

![Q图片2018060522051](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/aly_tomcat_11.png)

按如图填写

![Q图片2018060522062](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/aly_tomcat_12.png)

还要配置8080的端口

![Q图片2018060522072](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/aly_tomcat_13.png)

#### 重启tomcat

我们改动了配置文件需要重启tomcat才能生效，但是在linux上只能通过结束进程的方式关闭tomcat，我们得先找到tomcat的进程id，由于tomcat默认监听8080，我们可以通过这一点来找

```bash
netstat -anp | grep 8080
```

找到对应的进程编号，如图是我的进程编号，5881

![Q图片2018060522195](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/aly_tomcat_14.png)

结束5881就可以了

```bash
kill -9 5881
```

然后启动就可以了