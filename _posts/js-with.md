---
title: js-with
date: 2018-06-13 15:05:33
tags: [js，with]
categories: 薛玉博
description: js-with关键字
---



**一、基本说明**

在js高级程序设计中是这样描述with关键字的：with语句的作用是将代码的作用域设置到一个特定的作用域中，基本语法如下：

```
with (expression) statement;
```

使用with关键字的目的是为了简化多次编写访问同一对象的工作，比如下面的例子：

```
var qs = location.search.substring(1);
var hostName = location.hostname;
var url = location.href;
```

这几行代码都是访问location对象中的属性，如果使用with关键字的话，可以简化代码如下：

```
with (location){
  var qs = search.substring(1);
  var hostName = hostname;
  var url = href;
}
```

在这段代码中，使用了with语句关联了location对象，这就以为着在with代码块内部，每个变量首先被认为是一个局部变量，如果局部变量与location对象的某个属性同名，则这个局部变量会指向location对象属性。
注意：在严格模式下不能使用with语句。

**二、with关键字的弊端**

前面的基本说明中，我们可以看到with的作用之一是简化代码。但是为什么不推荐使用呢？下面我们来说说with的缺点：

1、性能问题
2、语义不明，调试困难

**三、性能问题**

首先说说性能问题，关于使用with关键字的性能问题，首先我们来看看两段代码：

第一段代码是没有使用with关键字：

```
function func() {
  console.time("func");
  var obj = {
    a: [1, 2, 3]
  };
  for (var i = 0; i < 100000; i++) {
    var v = obj.a[0];
  }
  console.timeEnd("func");//0.847ms
}
func();
```

第二段代码使用了with关键字：

```
function funcWith() {
  console.time("funcWith");
  var obj = {
    a: [1, 2, 3]
  };
  var obj2 = { x: 2 };
  with (obj2) {
    console.log(x);
    for (var i = 0; i < 100000; i++) {
      var v = obj.a[0];
    }
  }
  console.timeEnd("funcWith");//84.808ms
}
funcWith();
```

在使用了with关键字后了，代码的性能大幅度降低。第二段代码的with语句作用到了obj2这个对象上，然后with块里面访问的却是obj对象。有一种观点是：使用了with关键字后，在with块内访问变量时，首先会在obj2上查找是否有名为obj的属性，如果没有，再进行下一步查找，这个过程导致了性能的降低。但是程序性能真正降低的原因真的是这样吗？

我们修改一下第二段代码，修改如下：

```
function funcWith() {
  console.time("funcWith");
  var obj = {
    a: [1, 2, 3]
  };
  with (obj) {
    for (var i = 0; i < 100000; i++) {
      var v = a[0];
    }
  }
  console.timeEnd("funcWith");//88.260ms
}
funcWith();
```

这段代码将with语句作用到了obj对象上，然后直接使用a访问obj的a属性，按照前面说到的观点，访问a属性时，是一次性就可以在obj上找到该属性的，但是为什么代码性能依旧降低了呢。
真正的原因是：使用了with关键字后，JS引擎无法对这段代码进行优化。
JS引擎在代码执行之前有一个编译阶段，在不使用with关键字的时候，js引擎知道a是obj上的一个属性，它就可以静态分析代码来增强标识符的解析，从而优化了代码，因此代码执行的效率就提高了。使用了with关键字后，js引擎无法分辨出a变量是局部变量还是obj的一个属性，因此，js引擎在遇到with关键字后，它就会对这段代码放弃优化，所以执行效率就降低了。
使用with关键字对性能的影响还有一点就是js压缩工具，它无法对这段代码进行压缩，这也是影响性能的一个因素。

**四、语义不明，难以调试**

前面说到除了性能的问题，with还存在的一个缺点语义不明，难以调试，就是造成代码的不易阅读，而且可能造成潜在的bug。

```
function foo(obj) {
  with (obj) {
    a = 2;
  }
}
var o1 = {
  a: 3
};
var o2 = {
  b: 3
};
foo(o1);
console.log(o1.a); // 2
foo(o2);
console.log( o2.a ); // undefined
console.log( a ); // 2
```

这段代码很容易理解了，在foo函数内，使用了with关键字来访问传进来的obj对象，然后修改a属性。当传入o1对象时，因为o1对象存在着a属性，所以这样没有问题。传入o2对象时，在修改a属性时，由于o2对象没有a这个属性，所以被修改的a属性则变成了全局变量。这就造成了潜在的bug。

**五、总结**

总的来说，强烈不推荐使用with关键字。其实在日常编码中，我们只需要知道不去使用with就可以了。



