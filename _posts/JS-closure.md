---
title: JS-closure
date: 2018-05-25 09:35:06
tags: [JavaScript，closure]
categories: 卢文才
description: JS闭包
---

### 1，什么是闭包（closure）

```
function external()
{ 
   var N = 0; // N是external函数的局部变量 
   function inside() // inside是external函数的内部函数，是闭包 
    { 
        N += 1; // 内部函数inside中使用了外部函数external中的变量N
        console.log(N);
    }
    return inside; 
} 
var result = external();
result(); // 输出1
result(); // 输出2 
result(); // 输出3
```

函数external的内部函数inside被函数external外的一个变量 result 引用。把这句话再加工一下就变成了闭包的定义，当一个内部函数被其外部函数之外的变量引用时，就形成了一个闭包。闭包就是外部函数返回之后，内部函数依然可以访问外部函数内的变量。

### 2，闭包的作用，实例

设想下如果你想统计一些数值，且该计数器在所有函数中都是可用的。

你可以使用全局变量，函数设置计数器递增：

## 实例

```
var counter = 0;

function add() {

    counter += 1;

}

add();

add();

add();

// 计数器现在为 3

```

计数器数值在执行 add() 函数时发生变化。

但问题来了，页面上的任何脚本都能改变计数器，即便没有调用 add() 函数。

如果我在函数内声明计数器，如果没有调用函数将无法修改计数器的值：

## 实例

```
function add() {

    var counter = 0;

    counter += 1;

}

add();

add();

add();

// 本意是想输出 3, 但事与愿违，输出的都是 1 !

```

以上代码将无法正确输出，每次我调用 add() 函数，计数器都会设置为 1。

**JavaScript 内嵌函数可以解决该问题。**

------

## JavaScript 内嵌函数

所有函数都能访问全局变量。  

实际上，在 JavaScript 中，所有函数都能访问它们上一层的作用域。

JavaScript 支持嵌套函数。嵌套函数可以访问上一层的函数变量。

该实例中，内嵌函数 **plus()** 可以访问父函数的 **counter** 变量：

## 实例

```
function add() {

    var counter = 0;

    function plus() {counter += 1;}

    plus();    

    return counter; 

}

```

如果我们能在外部访问 **plus()** 函数，这样就能解决计数器的困境。

我们同样需要确保 **counter = 0** 只执行一次。

**我们需要闭包。**

------

## JavaScript 闭包

还记得函数自我调用吗？该函数会做什么？

## 实例

```
var add = (function () {

    var counter = 0;

    return function () {return counter += 1;}

})();

add();

add();

add();
// 计数器为 3
```

变量 **add** 指定了函数自我调用的返回字值。

自我调用函数只执行一次。设置计数器为 0。并返回函数表达式。

add变量可以作为一个函数使用。非常棒的部分是它可以访问函数上一层作用域的计数器。

这个叫作 JavaScript **闭包。**它使得函数拥有私有变量变成可能。

计数器受匿名函数的作用域保护，只能通过 add 方法修改。