---
title: Github-Rss
date: 2018-04-25 10:38:13
tags: [Rss]
categories: 薛玉博
description: rss

---

### 1，前言

​	主要写一下配置Rss的过程及遇到的问题，解决方案。

### 2，安装过程

​	hexo 有对应相关的[插件](https://github.com/hexojs/hexo-generator-feed)。输入以下命令安装插件：

​	`npm install hexo-generator-feed --save`

​	安装完成之后，在博客的配置文件      `_config.yml` 里添加如下配置：

```
feed:
  type: atom
  path: atom.xml
  limit: 20
  hub:
  content:
  content_limit: 140
  content_limit_delim: ' '
```

​	编译之后，`public` 根目录下会生成 `atom.xml` 文件，这就说明成功了。另     	外，一般主题都会默认配置好 RSS 文件的路径，如果没有，则在主题的配置文件里填上即可。

### 3，安装遇到的问题

​	我在按照上述步骤安装成功之后，编译生成了     `atom.xml` 文件，主题里也配置了RSS路径，但我打开rss之后，就报错找不到  `atom.xml`文件：

​	![](\xueyb0911.github.io\xueyb\atom_err.png)

### 4，原因

​	是因为我改了博客配置文件      `_config.yml` 里的url和root属性：

​		![](\xueyb0911.github.io\xueyb\yml.png)

​	导致配置的Rss路径找不到文件。

### 5，解决方案

​	可以把博客配置文件      `_config.yml` 里的url和root属性改回原来的：

​		![](\xueyb0911.github.io\xueyb\yml2.png)

​	也可以在主题的配置文件里改变Rss配置的路径：

​		![](\xueyb0911.github.io\xueyb\themes_yml.png)