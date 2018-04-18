---
title:  处理hexo使用next主题遇到的问题
date: 2018-04-17 10:42:00 
tags: [hexo] 
categories: 甘明阳 
description: 主要是记录要改动的配置信息
---



ps: 之前用hexo写的博客一直有问题，首先是样式加载不到，我将hexo文件夹下的_config.yml里面的relative_link改为true之后就可以正常加载了，本来以为问题已经解决，但是替换next主题后陆陆续续各种路径找不到，索性就将hexo卸载重装，用了最新版本hexo后感觉世界都变得美好了，重新提交了blog，现在记录一下相关的配置。<!--more-->



### 重装hexo

卸载hexo，3.0.0之前版本执行

```bash
npm uninstall hexo -g
```

3.0.0之后版本执行

```bash
npm uninstall hexo -g
```

查看hexo版本

```bash
hexo -v
```

安装最新版hexo

```bash
npm install hexo-cli -g
```

### next主题安装

1、下载next主题

​     进入你的博客下的themes文件夹下，目录结构类似Hexo\blog\themes，输入

```bash
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

​     会下载一个next文件夹，如图所示

![exo_themes_0](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/hexo_themes_01.png?raw=true)

  2、修改主题

​        找到Hexo\blog\\_config.yml，将theme原来的landscape改为next,如图

![exo_themes_0](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/hexo_themes_02.png?raw=true)

​        重启服务，在git里面输入

```bash
hexo s
```



### 修改配置文件

​      1、配置语言

​           现在你的主题就成功改好了，但是还有一些小小的问题，我们找到Hexo\blog\themes\next\_config.yml，搜索

title，修改如下:

```bash
# Site
title: 小小程序员
subtitle: 记录学习过程中的点点滴滴
description: 技术与生活，与我融为一体
keywords: linux
author: 甘明阳
language: zh-Hans
timezone: 
```

​           

然后我们去Hexo\blog\themes\next\_config.yml，搜索scheme，如图：

![exo_themes_0](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/hexo_themes_04.png?raw=true)

这里有四种样式可以供我们选择，记住，有一个要去掉#，然后其他三个要加上#

​      2、添加头像

​           我们要加上自己的头像，在这个文件里搜索avatar，如图：

![exo_themes_0](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/hexo_themes_05.png?raw=true)

​           将avatar的值改为我们自己的图片路径，我的路径是     

```bash
https://raw.githubusercontent.com/ganmyds/markdown_img_test/master/img/head.jpg
```

![exo_themes_0](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/hexo_themes_06.png?raw=true)

​      3、添加社交链接

​           还是在当前文件，搜索social，添加如下：

```bash
social:
  github: https://github.com/your-user-name
  twitter: https://twitter.com/your-user-name
  weibo: http://weibo.com/your-user-name
  douban: http://douban.com/people/your-user-name
  zhihu: http://www.zhihu.com/people/your-user-name
```

​           内容可根据需求增减

---

### 处理一些next的问题

问题一、分类和标签按钮不能点击

​    next目前没有分类和标签页面，需要我们手动添加，首先添加分类页面，在git命令行里面输入：

```bash
hexo new page categories
```

完成后在source里面生成一个categories文件夹，我们进入Hexo\blog\source\categories，找到index.md，改动如下：

```bash
title: categories
date: 2018-04-17 15:47:33
type: "categories"
```

然后在git命令行里面输入:

```bash
hexo g
```

刷新页面，分类按钮就能点击了。

然后添加标签页面，输入：

```bash
hexo new page tag
```

进入Hexo\blog\source\tag，找到index.md,改动如下

```bash
title: categories
date: 2018-04-17 15:47:33
type: "tags"
```

剩下的步骤同上。

---

问题二、点击首页回不去

   这个要看你的博客是不是处于子目录里面，例如我的博客路径为

```bash
 https://ganmyds.github.io/hexo_blog/
```

  就是有子目录,下面就是不含子目录

```bash
https://ganmyds.github.io/
```

如果你是将博客放在子目录里面，找到Hexo\blog\themes\next\_config.yml，搜索menu内容类似下面：

```bash
menu:
  home: /
  archives: /archives
  #about: /about
  #categories: /categories
  tags: /tags
  #commonweal: /404.html
```

将内容改为你博客的对应链接就可以了，例如我的改动是：

```bash
menu:
  home: https://ganmyds.github.io/hexo_blog/
  # about: /about/ || user
  tags: https://ganmyds.github.io/hexo_blog/tags/ || tags
  categories: https://ganmyds.github.io/hexo_blog/categories/ || th
  archives: https://ganmyds.github.io/hexo_blog/archives/ || archive
  # schedule: schedule/ || calendar
  # sitemap: sitemap.xml || sitemap
  # commonweal: 404/ || heartbeat
```



   