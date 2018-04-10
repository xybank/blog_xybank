---
title: React 简单介绍
date: 2018-04-09 11:30:00
tags: [react]
categories: 李志丹
---
React 是较早使用 JavaScript 构建大型、快速的 Web 应用程序的技术方案。它已经被广泛应用于 Facebook 和 Instagram。
<!--more-->
## React介绍

React 是为了解决一个问题：构建随着时间数据不断变化的大规模应用程序。

React 采用下面两个主要的思想：

- 简单：表达出应用程序在任一个时间点应该长的样子，然后当底层的数据变了，React 会自动处理所有用户界面的更新。
- 声明式：当数据变化了，React 概念上是类似点击了更新的按钮，但仅会更新变化的部分。

React 都是关于构建`可复用的组件`。通过 React 唯一要做的事情就是构建组件。得益于其良好的封装性，组件使代码复用。

## 背景

目前Web应用的开发流程中，前端代码的开发和维护的工作量日益增大。与此同时，开源社区中也涌现出了很多的前端开发框架用于提升前端代码开发和维护的效率。按照类型分为：

- UI框架。如Bootstrap、MUI等
- Library类型框架。如JQuery等
- 架构类框架。如AngularJS、Backbone、React

使用统一的开发框架、开发规范无疑可以给MUIP上应用开发的效率和维护成本带来好处。结合移动应用的开发特点，框架需要满足以下的要求：

- 提供公共组件的封装机制，较容易实现组件及代码复用；
- 满足移动设备上的运行效率要求，较容易实现好的用户体验；
- 开放标准及开发源码，有良好的可扩展性和可集成性；
- 社区活跃，有大量的开发人群；

综合以上考虑，我们拟选择React框架来为作为应用开发的默认框架，并基于它定义应用上的公共组件。

## Webpack介绍

Webpack 是一个模块打包工具，可以使用Webpack管理模块依赖，并编绎输出模块所需的静态文件。

Webpack优点：

- 模块来源广泛，支持包括npm/bower等等的各种主流模块安装／依赖解决方案。
- 模块规范支持全面，amd/commonjs以及shimming等一应具全。
- 浏览器端足迹小，移动端友好，却对热加载乃至热替换有很好的支持。
- 插件机制完善，实现本身实现同样模块化，容易扩展。

支持的两种模块加载模式：

- 同步加载模式： CommonJS (Node.JS)的模式
- 异步加载模式：即 AMD 模式，与require.js相同

## 为何使用Webpack

长久以来，Web开发者都是把所需Javascript、CSS文件一并放进HTML里面，对于庞大的项目来说管理起来非常麻烦。

Webpack就是一款专为Web开发设计的包管理器。它能够很好地管理、打包Web开发中所用到的Javascript、CSS以及各种静态文件（图片、字体等），等资源文件会被编译整合到一个静态文件中，对于开发人员只需简单引用这一个静态文件即可，让开发过程更加高效。

对于应用组件开发，将遵循CommonJS的开发规范，对组件进行开发，使用Webpack对所有的组件进行打包，从而避免开发人员每个组件单独的引入，对开发人员还可忽略复合组件间的依赖关系，无论对后期开发还是维护，对开发效率的提升上，都起到了关键的作用

# 环境安装

## Node.js介绍

Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境。Node.js 使用了一个事件驱动、非阻塞式 I/O 的模型，使其轻量又高效。Node.js 的包管理器 `npm`，是全球最大的开源库生态系统。

在基于React的开发中正正使用到了Node.js的包管理工具`npm`，对资源包进行有效地管理。

## Node.js安装

**第一步：下载安装文件**

下载nodejs，官网：<http://nodejs.cn/download/>，下载node-v5.3.0-x64.msi(Mac系统下载node-v5.3.0.pkg)，如下图：
![![img](/img/lizd/node_download.png)](/img/lizd/node_download.png)

**第二步：Node.js安装**

下载完成之后，双击"node-v5.3.0-x64.msi"，开始安装nodejs，默认安装在`C:\Program Files\nodejs\`下。

在cmd控制台输入：

```
node -v
```

控制台将打印出：`v8.9.3`，出现版本提示表示安装成功。

新版的Nodejs已经集成了`npm`，所以之前npm也一并安装好了。同样可以使用cmd命令行输入

```
npm -v
```

控制台将打印出：`5.8.0`，出现版本提示表示安装成功。

## 什么是Webpack

![![img](/img/lizd/what-is-webpack.png)](/img/lizd/what-is-webpack.png)

事实上它是一个打包工具，而不是像RequireJS或SeaJS这样的模块加载器，通过使用Webpack，能够像Node.js一样处理依赖关系，然后解析出模块之间的依赖，将代码打包。

Webpack可把各种资源，例如JS、JSX、coffee、样式（含less/sass）、图片等都作为模块来使用和处理。可以直接使用 require(xxx) 的形式来引入各模块，即使它们可能需要经过编译（比如JSX和sass），但无须在上面花费太多心思，因为 webpack 有着各种健全的加载器（loader）在默默处理这些事情。

## Webpack安装

全局安装

```
npm install -g webpack
```

