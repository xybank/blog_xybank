---
title: Android MVP设计架构学习
date: 2018-04-11 15:05:00
tags: [Android,MVP,设计架构]
description: 本文只研究MVP设计架构基础版，后续由各大神扩展的升级版以后研究
---

## MVC缺点
在Android开发中，Activity并不是一个标准的MVC模式中的Controller，它的首要职责是加载应用的布局和初始化用户 界面，并接受并处理来自用户的操作请求，进而作出响应。随着界面及其逻辑的复杂度不断提升，Activity类的职责不断增加，以致变得庞大臃肿。

## 什么是MVP
MVP从更早的MVC框架演变过来，与MVC有一定的相似性：Controller/Presenter负责逻辑的处理，Model提供数据，View负责显示。
![](http://assets.tianmaying.com/md-image/ea995e88af236afbd8fdc4906a67e829)

MVP框架由3部分组成：View负责显示，Presenter负责逻辑处理，Model提供数据。在MVP模式里通常包含3个要素（加上View interface是4个）：

View:负责绘制UI元素、与用户进行交互(在Android中体现为Activity)

Model:负责存储、检索、操纵数据(有时也实现一个Model interface用来降低耦合)

Presenter:作为View与Model交互的中间纽带，处理与用户交互的负责逻辑。

*View interface:需要View实现的接口，View通过View interface与Presenter进行交互，降低耦合，方便进行单元测试

## MVC → MVP
当我们将Activity复杂的逻辑处理移至另外的一个类（Presenter）中时，Activity其实就是MVP模式中的View，它负责UI元素的初始化，建立UI元素与Presenter的关联（Listener之类），同时自己也会处理一些简单的逻辑（复杂的逻辑交由 Presenter处理）。

MVP的Presenter是框架的控制者，承担了大量的逻辑操作，而MVC的Controller更多时候承担一种转发的作用。因此在App中引入MVP的原因，是为了将此前在Activty中包含的大量逻辑操作放到控制层中，避免Activity的臃肿。

两种模式的主要区别：

（最主要区别）View与Model并不直接交互，而是通过与Presenter交互来与Model间接交互。而在MVC中View可以与Model直接交互
通常View与Presenter是一对一的，但复杂的View可能绑定多个Presenter来处理逻辑。而Controller是基于行为的，并且可以被多个View共享，Controller可以负责决定显示哪个View
Presenter与View的交互是通过接口来进行的，更有利于添加单元测试。
MVC与MVP区别

因此我们可以发现MVP的优点如下：

1、模型与视图完全分离，我们可以修改视图而不影响模型；

2、可以更高效地使用模型，因为所有的交互都发生在一个地方——Presenter内部；

3、我们可以将一个Presenter用于多个视图，而不需要改变Presenter的逻辑。这个特性非常的有用，因为视图的变化总是比模型的变化频繁；

4、如果我们把逻辑放在Presenter中，那么我们就可以脱离用户接口来测试这些逻辑（单元测试）。

原文参考链接：[https://blog.csdn.net/lovingkid/article/details/50917907](https://blog.csdn.net/lovingkid/article/details/50917907)



## 总结
MVP设计架构比MVC相对复杂，但有益于降低UI、逻辑、业务、数据之间的耦合度，方便代码扩展及维护性。基于该原因，练手项目Android决定采用MVP设计架构，同时也方便后续练手项目的复用。

MVP设计架构github地址：[https://github.com/flsboy/mvpDemo.git](https://github.com/flsboy/mvpDemo.git)