---
title: 用Jenkins自动部署hexo搭建的GitHub静态博客
date: 2018-04-02 13:30:00
tags: [hexo, Jenkins]
categories: 蔡向炜
---

hexo 为我们提供了一套非常方便的静态博客搭建框架，我们只需要编写文章的 MarkDown 文件即可。但是即便方便，还是会有几个繁琐的步骤：每次写一篇博客都需要执行编译命令，再将新生成的网页代码提交到 GitHub；而且，如果博客是由团队成员共同贡献，很难做到协同维护。使用 Jenkins 可以帮我们一定程度上解决这两个问题。

<!--more-->

## 一、基本思路

Jenkins 是一套非常常用且好用的自动化构建工具，也为我们提供了很多插件。这其中会用到 GitHub 提供的 WebHook 功能，GitHub 收到一个 Push 后，会向 Jenkins 服务器发起一条 Post 请求，告诉 Jenkins 现在可以去执行部署操作了。

## 二、安装 Jenkins

Jenkins 的安装过程可以完全参考官网：[Jenkins 官网](https://wiki.jenkins.io/display/JENKINS/Installing+Jenkins)，不同系统有不同的安装方法，比如 CentOS：

![Jenkins_1](/img/hexo/Jenkins_1.png)

只要按照步骤安装即可，基本上都会包含以下步骤：

1. 安装 Jenkins，一般安装稳定版会比较稳妥；
2. 安装 Java 环境，Jenkins 是需要 Java 环境的；
3. 启动 Jenkins 服务，设置开机自启动；
4. 关闭相应的防火墙；

安装完成后，访问 `http://你的服务器ip:8080/` 即可，Jenkins 默认端口是8080，有些云服务器会限制端口的访问，这时候需要在控制台修改配置，比如阿里云是这样的：

![Jenkins_2](/img/hexo/Jenkins_2.png)

安装好 Jenkins 后，第一次访问会让你输入密码，页面上会提示具体是哪个文件，用 vim 打开即可查看，比如 CentOS 默认是：`/var/lib/jenkins/secrets/initialAdminPassword`。

进入后，可以创建新的用户，以后就可以使用这个用户访问 Jenkins 了。

新建账号后，会提示安装插件，一般直接按推荐安装，如果有报错，最好点击重试，多试几次一般都会安装成功，如果实在装不上那就先跳过吧

![Jenkins_3](/img/hexo/Jenkins_3.png)

## 三、在服务器上安装 hexo

在服务器上安装 hexo，是为了可以让 Jenkins 自动部署。安装方法参考之前的文章：[基于hexo搭建GitHub静态博客](/2018/03/23/blog_hexo_GitHub/)。Linux 肯定会不一样，请自行百度/谷歌解决。为了拉/上传代码，还需要安装 Git。

安装好 hexo 后，后面构建生成静态网站、上传网页新代码等，都要在这里做了。所以你需要做的有：

1. 初始化一套和你原来博客一样的配置，包括配置和主题；
2. 从你的 Git 上拉一份你博客网页（xxx.github.io）项目的代码，用于提交新代码；
3. 新建一个只存放 hexo 的 source 文件夹的库，就是 hexo 中放 md、img 等源文件的文件夹，这里暂时称它为 `blog_source`；

## 四、配置 Jenkins 自动构建项目

自动构建是基于上一节中新建的 `blog_source` 项目进行的。

新建任务，输入任务名，选择自由风格即可：

![Jenkins_4](/img/hexo/Jenkins_4.png)

配置项目，设置 GitHub 项目地址：

![Jenkins_5](/img/hexo/Jenkins_5.png)

源码管理，这里一定要注意，设置为你的 `blog_source` 的 GitHub 源码 Url，否则 Jenkins 收到 GitHub 的 Post 请求的时候，是不知道你到底要执行那个任务的：

![Jenkins_6](/img/hexo/Jenkins_6.png)

构建触发器，选 GitHub hook：

![Jenkins_7](/img/hexo/Jenkins_7.png)

构建，这里就是真正的构建步骤，可以按你的需求填。我这边全部用 shell 命令执行：

![Jenkins_8](/img/hexo/Jenkins_8.png)

配置好后，点击应用即可。

注意：如果构建的过程中提示没有权限之类的错误，请设置一下 Jenkins 用户。Jenkins 安装好后，会自动新建一个用户叫 jenkins，可以把它改为 root 用户。 CentOS 配置文件为 `/etc/sysconfig/jenkins` ，修改 JENKINS_USER 项为 root 即可：

```
JENKINS_USER="root"
```

另外，还需要配置 Jenkins 的 GitHub 插件的 hook url：

![Jenkins_9](/img/hexo/Jenkins_9.png)

![Jenkins_10](/img/hexo/Jenkins_10.png)

![Jenkins_11](/img/hexo/Jenkins_11.png)

## 五、 配置 Github 项目

配置 `blog_source` 项目的 Webhooks ，一定要以 `/` 结尾：

![Jenkins_12](/img/hexo/Jenkins_12.png)

## 六、配置结束

完成以上配置后，就可以试试向 `blog_source` push 代码了，如果 Jenkins 在构建了，就说明配置成功了，等构建完成，就可以预览你的博客了。
 