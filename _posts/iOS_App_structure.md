---
title: iOS App架构初研
date: 2018-03-30 16:00:00
tags: [iOS] 
categories: 洪宇杰 
---

> 目前市场上iOS平台的APP架构最流行的主要分两种：MVC和MVVM
> 本次就对这两种架构进行初步研究
> 参考资料:[杂谈: MVC/MVP/MVVM](https://www.jianshu.com/p/eedbc820d40a)

<!--more-->

## MVC
![MVC](http://cc.cocimg.com/api/uploads/20160107/1452152304301101.png)
MVC，Model-View-Controller，我们从这个古老而经典的设计模式入手。采用 MVC 这个架构的最大的优点在于其概念简单，易于理解。而在 iOS 客户端开发中，MVC 作为官方推荐的主流架构，不但 SDK 已经为我们实现好了 UIView、UIViewController 等相关的组件，更是有大量的文档和范例供我们参考学习，可以说是一种非常通用而成熟的架构设计。

但 MVC 也有他的坏处。由于 MVC 的概念过于简单朴素，已经越来越难以适应如今客户端的需求，大量的代码逻辑在 MVC 中并没有定义得很清楚究竟应该放在什么地方，导致他们很容易就会堆积在 Controller 里，成为了人们所说的 Massive View Controller（重量级视图控制器）。

## MVVM
![](http://cc.cocimg.com/api/uploads/20160107/1452153249521047.png)
MVVM，Model-View-ViewModel，一个从 MVC 模式中进化而来的设计模式，最早于2005年被微软的 WPF 和 Silverlight 的架构师 John Gossman 提出。在 iOS 开发中实践 MVVM 的话，通常会把大量原来放在 ViewController 里的视图逻辑和数据逻辑移到 ViewModel 里，从而有效的减轻了 ViewController 的负担。。MVVM 通常还会和一个强大的绑定机制一同工作，一旦 ViewModel 所对应的 Model 发生变化时，ViewModel 的属性也会发生变化，而相对应的 View 也随即产生变化。  
同样的，MVVM 也有他的缺点：

一个首要的缺点是，MVVM 的学习成本和开发成本都很高。MVVM 是一个年轻的设计模式，大多数人对他的了解都不如 MVC 熟悉，基于绑定机制来进行编程需要一定的学习才能较好的上手。同时在 iOS 客户端开发中，并没有现成的绑定机制可以使用，要么使用 KVO，要么引入类似 ReactiveCocoa 这样的第三方库，使得学习成本和开发成本进一步提高。

另一个缺点是，数据绑定使 Debug 变得更难了。数据绑定使程序异常能快速的传递到其他位置，在界面上发现的 Bug 有可能是由 ViewModel 造成的，也有可能是由 Model 层造成的，传递链越长，对 Bug 的定位就越困难。

同时还必须指出的是，在传统的 MVVM 架构中，ViewModel 依然承载的大量的逻辑，包括业务逻辑，界面逻辑，数据存储和网络相关，使得 ViewModel 仍然有可能变得和 MVC 中 ViewController 一样臃肿。  

## 总结
MVVM是为了解决MVC中C层的代码堆积的问题，把MVC中C层拆分成ViewControl和ViewModel。但是相应的如果使用MVVM架构开发成本较高，需要引入绑定机制来进行编程（ReactiveCocoa）。考虑到我们练手项目的复杂程度不高，C层的代码堆积问题不是很严重，建议使用MVC架构。


