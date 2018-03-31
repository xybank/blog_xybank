---
title:  linux上面安装配置mysql
date: 2018-03-29 19:05:00 
tags: [linux, mysql] 
categories: 甘明阳 
description: linux上搭建mysql流程 
---


一个项目没有数据库可以说是无法想象的，作为一名开发人员，我们需要经常性的操作数据库。现在市面上有很多数据库可供我们使用，比如ACCESS，MSSQL，MYSQL，ORACLE，DB2,其中mysql可以说是目前最流行的数据库之一，mysql数据库主要有以下几个好处：<!--more-->

* 1. 免费
  2. 跨平台
  3. 轻量级
  4. 支持多并发

mysql只有在linux运行环境下才能充分发挥其威力，在linux环境下搭建mysql就变得非常有意义了，下面我为大家详细介绍linux下mysql的搭建

1、下载mysql安装文件，有两个文件，分别是mysql-connector-java-5.1.6-bin.jar和mysql-standard-4.0.26-pc-  linux-gnu-i686.tar.gz，点击下载https://pan.baidu.com/s/1es5NjK-VOiohtkScZKt6iw

将两个安装文件拷贝到/home下,切换到root，输入命令：

su root

2、解压安装文件:

tar -zxvf  mysql-standard-4.0.26-pc-linux-gnu-i686.tar.gz

解压缩后我们会得到一个目录如图

![mg0](\img\ganmingyang\linux_1.png)



3、创建一个文件夹mysql专门用于存放mysql文件,输入命令 :

​     mkdir mysql

4、将目录mysql-standard-4.0.26-pc-linux-gnu-i686下的文件移动到mysql里面，输入命令:

​    mv mysql-standard-4.0.26-pc-linux-gnu-i686 mysql

5、创建一个组mysql,注意这个命令要root权限才能使用，专门创建mysql组是为了方便以后管理mysql下面   的各个用户，输入命令:

​     groupadd mysql

6、创建mysql用户放入mysql组，输入命令:

​    useradd -g mysql mysql

7、进入mysql目录，输入命令 cd mysql

8、初始化mysql数据库，输入命令：

​    scripts/mysql_install_db  --user=mysql 如图就是初始化成功：

![mg0](\img\ganmingyang\linux_5.png)

9、将mysql目录下所有文件以及子文件的所有者改为root、所在组改为mysql，输入命令：

  chown -R root:mysql .

10、将数据文件夹用户改为mysql,输入命令：chown -R mysql data,我们检查一下是否成功改动，输入

   命令: ls -l ,结果如图所示：

![mg0](\img\ganmingyang\linux_4.png)

11、后台运行mysql,输入命令： 

  bin/mysqld_safe   --user=mysql & ,结果如图所示：

![mg0](\img\ganmingyang\linux_3.png)

12、看看我们的mysql是否正常运行，输入命令netstat -anp | moe，看看3306端口是否存在，如图：

![mg0](\img\ganmingyang\linux_2.png)

13、进入bin目录,输入命令:cd bin

14、运行mysql,输入命令：

​     mysql -u root -p，

Enter password是要你输入密码，目前没有密码直接按enter即可，如图:

​              ![mg0](\img\ganmingyang\linux_6.png) 

现在mysql已经成功的安装到我们的linux下面了。到这里基本完成了，如果你想让以后的工作更方便，看看下面的内容。



***



现在我们打开mysql非常麻烦，需要进入/home/mysql/bin目录下执行，为了减少麻烦，我们可以将这个路径配置到环境变量里面，输入命令 

​    cd /root 

​    vi  .bash_profile 

改动如图

![mg0](\img\ganmingyang\linux_7.png)

然后我们登出让配置文件生效，输入命令logout ，重新登录，现在你就可以在任意路径上运行mysql了。