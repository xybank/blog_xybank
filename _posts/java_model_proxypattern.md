---
title: 设计模式——代理模式
date: 2018-05-31
tags: [java,设计模式]
categories: 童长钱
description: Java设计模式之代理模式，简单来说就是interface（接口的使用）
---

## 代理模式【 Proxy Pattern】

#### 一、什么是代理模式
[百度说明](https://baike.baidu.com/item/%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F/8374046?fr=aladdin)：
组成：（抽象角色：通过接口或抽象类声明真实角色实现的业务方法。
代理角色：实现抽象角色，是真实角色的代理，通过真实角色的业务逻辑方法来实现抽象方法，并可以附加自己的操作。真实角色：实现抽象角色，定义真实角色所要实现的业务逻辑，供代理角色调用。）


**通俗的来说呢**：A要完成一些事情，但事实上他很忙，没有时间完成或者....找一个可靠的代理人B，所有要完成的任务委托给B，这C去找能够完成这些任务的人C1,C2,C3,C4...(相当于现实生活中的中介，代理律师等)，在现实生活正中各种服务型App就是一种代理。。。


生活中处处是代理。

<!--more-->

#### 二、代理模式设计过程

1. **代理接口**
```
public interface  AllDeputize {
2     void AllFunction()
5     }
6 }
```

2. **代理类**

```
public class Agent implements AllDeputize {
2     @Override
3     public void AllFunction() {
4         new Consigner().AllFunction();
5     }
6 }
```

3. **委托类**
```
 public class Consigner implements AllDeputize {
 2     public void goShopping() {
 3         System.out.println("买东西");
 4     }
 5     public void party() {
 6         System.out.println("参加party");
 7     }
 8     @Override
 9     public void AllFunction() {
10         System.out.println("代理事物");
11     }
12 }
```

4. **测试代理**
```
public class Clienter {
    public static void main(String[] args) {
        AllDeputize ad = new Agent();
        ad.Quanli();
    }
}
```

5. **总结**

代理模式很简单，只要记住以下关键点，简单易实现：

　　　　（1）代理类与委托类实现同一接口

　　　　（2）在委托类中实现功能，在代理类的方法中中引用委托类的同名方法

　　　　（3）外部类调用委托类某个方法时，直接以接口指向代理类的实例，这正是代理的意义所在：屏蔽。

　　代理模式场景描述：

　　　　（1）当我们想要隐藏某个类时，可以为其提供代理类

　　　　（2）当一个类需要对不同的调用者提供不同的调用权限时，可以使用代理类来实现（代理类不一定只有一个，我们可以建立多个代理类来实现，也可以在一个代理类中金进行权限判断来进行不同权限的功能调用）

　　　　（3）当我们要扩展某个类的某个功能时，可以使用代理模式，在代理类中进行简单扩展（只针对简单扩展，可在引用委托类的语句之前与之后进行）

　　代理模式虽然实现了调用者与委托类之间的强耦合，但是却增加了代理类与委托类之间的强耦合（在代理类中显式调用委托类的方法），而且增加代理类之后明显会增加处理时间，拖慢处理时间。