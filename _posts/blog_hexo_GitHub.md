---
title: 基于hexo搭建GitHub静态博客
date: 2018-03-23 19:05:00
tags: [hexo]
categories: 蔡向炜
description: 搭建博客参考教程
---

## 一.准备

[hexo](https://github.com/hexojs/hexo) 是一个工具，可以帮助我们自动生成博客网站的代码，我们只需要用 MarkDown 写好博客，再使用 hexo 生成网站代码，最后把网站代码部署到 GitHub 上即可。需要以下准备：

- 1.**GitHub**。 基于 Git 的代码托管网站，提供静态页面服务。首先要有一个GitHub账号，博客网站的代码是放在 GitHub 仓库上的。
- 2.**Git**。 需要安装 [Git](https://gitforwindows.org) 版本控制器，类似 SVN 的工具。
- 3.**Node.js**。 需要安装 [Node.js](http://nodejs.cn)。 hexo 是基于 nodejs 的一套框架工具。
- 4.**MarkDown**。 程序员最爱的写文档工具，写博客需要用到。具体语法可以参考CSDN的MarkDown语法模板 [下载](/download/MarkDown.rar)。

## 二.安装 hexo 环境。

安装好 Nodejs 后，就可以安装 hexo 框架了。

- 1.首先新建一个文件夹（涉及到代码的文件夹，请大家务必不要用中文），用于存放 hexo 框架，我这边以 `D:\work\hexo` 为例。

- 2.打开 CMD 命令窗口，cd 进入 `hexo` 文件夹，输入以下命令安装 hexo ：

``` bash
npm install hexo-cli -g
```

- 3.安装完成后，输入以下命令生成博客工程（文件夹），这边工程先命名为 `blog` ：

``` bash
hexo init blog
```

- 4.进入工程 `blog` 文件夹：

``` bash
cd blog
```

## 三.写文章

安装完成后，可以看到 `blog` 目录下有一堆文件和文件夹，

![Folder](/img/hexo/folder.jpg)

其中：

source 用于放 MarkDown 写的文章文档；

themes 是存放博客主题的文件夹；

public 文件夹中存放自动生成的网站代码。

知道这些就可以开始写博客了，可以看到 `\source\_posts` 下有一个默认的文档 helloworld ，就是一篇默认的博客。

- 1.新建一篇博客，在 `D:\work\hexo\blog` 目录下，输入以下命令会在 `D:\work\hexo\blog\source\_posts` 中生成一个 .md 后缀的 MarkDown 文件：

``` bash
hexo new "文章标题"
```

- 2.在生成的文件中写博客，一般的文本编辑器都可以编写 MarkDown 文档；

- 3.写完博客后，输入以下命令就可以在 `public` 目录下，自动生成静态的博客网站代码：

``` bash
hexo generate
或
hexo g
```

- 4.这时候启动本地服务，就可以预览网站（ http://localhost:4000/ ）效果了，启动服务命令：

``` bash
hexo server
```

## 四.部署到 GitHub 上

- 1.新建 GitHub 代码仓库，进入你的 [GitHub](https://github.com/) 主页，点击右上角的加号，选择 New repository 新建项目。这里要注意，代码仓库格式必须是： XXX.github.io ，其中 **XXX 必须是你 GitHub 的 ID**。

![GitHub](/img/hexo/GitHub_1.jpg)

![GitHub](/img/hexo/GitHub_2.jpg)

- 2.克隆 GitHub 代码仓库到本地，找个合适的文件夹放置网站代码：

``` bash
git clone https://github.com/XXX/XXX.github.io.git
```

https://github.com/XXX/XXX.github.io.git 是代码仓库地址，获取方式如下：

![GitHub](/img/hexo/GitHub_3.jpg)

- 3.上传博客网站代码。前面写完文章，运行命令生成的网站代码都在 public 文件夹中，复制 public 目录下的所有文件，粘贴到上一步的代码仓库（`XXX.github.io` 文件夹）中。提交代码，即在代码仓库文件夹中，右键 Git Bash Here，依次执行以下命令：

``` bash
git add . // 添加修改
git commit -m "备注" // 提交
git push origin master // 上传到GitHub代码库
```

然后输入账号密码，等待上传完成后，过几分钟就可以看到网站效果。

注意：如果你是首次使用 Git ，它会提示你设置一些配置，只需按提示输入配置命令即可。

### 以下是附加的，有兴趣可以尝试。

## 五.更换主题

以上步骤完成后，你就有一个博客了，但是还是默认的主题 landscape。可以在这两个地方找自己喜欢的主题：[hexo官网](https://hexo.io/themes/)、[有哪些好看的 Hexo 主题？](http://www.zhihu.com/question/24422335)，也可以直接上网搜 hexo 主题。

以此博客的主题 [Maupassant](https://github.com/tufu9441/maupassant-hexo) 为例：

- 1.克隆主题到 `themes` 文件夹中：

``` bash
git clone https://github.com/tufu9441/maupassant-hexo.git themes/maupassant
```

- 2.按照文档要求，安装插件：

``` bash
npm install hexo-renderer-pug --save
npm install hexo-renderer-sass --save
```

- 3.修改博客 hexo 目录下的 `_config.yml` 文件：将 theme 项的 landscape 改为 maupassant。这里提示下，`_config.yml` 中还有博客的其他配置，可以按需更改。

- 4.配置主题。主题文件夹中也有一个 `_config.yml` 文件，这个是主题里相关的配置，一般主题都会有配置文档可以参考。

当前博客的配置参考: [下载](/download/blog_onfig.rar)

## 六.RSS订阅

RSS就是提供接口出来，让别人可以订阅你的博客。hexo 有对应相关的[插件](https://github.com/hexojs/hexo-generator-feed)。输入以下命令安装插件：

``` bash
npm install hexo-generator-feed --save
```

在博客的配置文件 `_config.yml` 里添加如下配置，：

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

编译之后，`public` 根目录下会生成 `atom.xml` 文件，这就说明成功了。另外，一般主题都会默认配置好 RSS 文件的路径，如果没有，则在主题的配置文件里填上即可。

## 七.404网页

你还可以给自己自定义一个 404 网页，一旦访问你的网站的某个页面不存在时，就会跳到 404 网页。网上也有很多模板可以用，如：[腾讯的寻子公益404页面](http://www.qq.com/404/)。

在 source 下新建一个 404.html 或 404.md 文件，把以下代码复制进去：

```
<script type="text/javascript" src="http://www.qq.com/404/search_children.js" charset="utf-8" homePageUrl="/" homePageName="回到我的主页"></script>
``` 

两个参数"homePageUrl"、"homePageName"可以按需修改。
