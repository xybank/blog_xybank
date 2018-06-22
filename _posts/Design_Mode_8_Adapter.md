---
title: 设计模式学习系列（8）--适配器模式
date: 2018-06-22 10:30:00
tags: [设计模式]
categories: 蔡向炜
---

适配器模式：**将一个类的接口变换成客户端所需要的另一种接口，从而使原本因接口不匹配而无法一起工作的两个类能够在一起工作**

<!--more-->

![Adapter](http://www.laocaixw.cn/img/design_mode/8_Adapter.jpg)

客户端所需要的接口：

``` java
public interface Target {
    public void request();
}
```

原有的、需要被适配的类：

``` java
public class Adaptee {
    public void doSomething() {
        // 原有的业务逻辑
        System.out.println("doSomething");
    }
}
```

或对象适配器：

``` java
public class Adapter2 implements Target {
    private Adaptee adaptee;

    public Adapter2(Adaptee adaptee) {
        this.adaptee = adaptee;
    }

    @Override
    public void request() {
        adaptee.doSomething();
    }
}
```

客户端代码：

``` java
public class Demo {
    public static void main(String[] args) {
        Target target = new Adapter();
        // Target target = new Adapter2(new Adaptee());
        target.request();
    }
}
```

这里客户端所要用的是 Adaptee 的业务逻辑，但是需要的接口是 Target 类型的接口，原有的类 Adaptee 里提供的接口无法满足客户端的需求。所以这里就可以使用适配器 Adapter 来实现，根据 Target 的接口，来使用 Adaptee 的业务逻辑。


输出结果：

```
doSomething
```

### 适配器模式的使用场景

在已投产的系统中，当需要使用已有的类，但是这个类又不符合系统的接口时，使用适配器模式是最适合的，它可以将*不符合系统的接口的类*转换成*符合系统的接口*来使用。

《大话设计模式》中提到的一个典故：魏文王问扁鹊家中三兄弟谁的医术最好，扁鹊告诉魏文王，大哥最牛逼，然后是二哥，然后才是扁鹊。大哥治病都是在没发病之前就治好了，二哥是在刚发病的时候治好，而扁鹊病情严重的时候才治好的。这个故事告诉我们，适配器模式虽然好，但是最好还是在问题发生之前预防它，考虑全面，做好适配。

## 源码地址

https://github.com/laocaixw/DesignModeLearning
