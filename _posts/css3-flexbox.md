---
title: css3-flexbox（弹性盒子）
date: 2018-04-13 21:14:06
tags: [css3，flexbox]
categories: 卢文才
description: css3弹性盒子布局
---

### 1，flexbox简介

​          弹性盒子是 CSS3 的一种新的布局模式。CSS3 弹性盒（ Flexible Box 或 flexbox），是一种当页面需要适应不同的屏幕大小以及设备类型时确保元素拥有恰当的行为的布局方式。引入弹性盒布局模型的目的是提供一种更加有效的方式来对一个容器中的子元素进行排列、对齐和分配空白空间。

### 2，flexbox基本概念

采用 Flex 布局的元素，称为 Flex 容器（flex container），简称"容器"。它的所有子元素自动成为容器成员，称为 Flex 项目（flex item），简称"项目"。

![img](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071004.png)

容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做`main start`，结束位置叫做`main end`；交叉轴的开始位置叫做`cross start`，结束位置叫做`cross end`。

项目默认沿主轴排列。单个项目占据的主轴空间叫做`main size`，占据的交叉轴空间叫做`cross size`。

### 3，flexbox的使用

① 给父容器添加display: flex/inline-flex;属性，即可使容器内容采用弹性布局显示，而不遵循常规文档流的显示方式；

② 容器添加弹性布局后，仅仅是容器内容采用弹性布局，而容器自身在文档流中的定位方式依然遵循常规文档流；

③ display:flex; 容器添加弹性布局后，显示为块级元素；

display:inline-flex; 容器添加弹性布局后，显示为行级元素；

④ 设为 Flex布局后，子元素的float、clear和vertical-align属性将失效。但是position属性，依然生效。

### 4,flexbox特性

弹性容器外及弹性子元素内是正常渲染的。弹性盒子只定义了弹性子元素如何在弹性容器内布局。

弹性子元素通常在弹性盒子内一行显示。默认情况每个容器只有一行。以下元素展示了弹性子元素在一行内显示，从左到右:

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"> 
<title>flexbox</title> 
<style> 
.flex-container {
    display: -webkit-flex;
    display: flex;
    width: 400px;
    height: 250px;
    background-color: lightgrey;
}

.flex-item {
    background-color: cornflowerblue;
    width: 100px;
    height: 100px;
    margin: 10px;
}
</style>
</head>
<body>

<div class="flex-container">
  <div class="flex-item">flex item 1</div>
  <div class="flex-item">flex item 2</div>
  <div class="flex-item">flex item 3</div>  
</div>

</body>
</html>
```

![img](/img/luwencai/flex0.png)

盒子容器下的元素默认同行显示，即使元素为块级元素。而当容器内元素的宽度大于容器时，元素的宽度会自动收缩：

```
<style> 
.flex-container {
    display: -webkit-flex;
    display: flex;
    width: 400px;
    height: 250px;
    background-color: lightgrey;
}

.flex-item {
    background-color: cornflowerblue;
    width: 200px;
    height: 100px;
    margin: 10px;
}
</style>
<div class="flex-container">
  <div class="flex-item">flex item 1</div>
  <div class="flex-item">flex item 2</div>
  <div class="flex-item">flex item 3</div>  
</div>
```

![img](/img/luwencai/flex1.png)

但是元素不是无限收缩的，当元素宽度收缩到元素的第一行文本宽度时，元素不再收缩。此时，若元素宽度依然大于容器，则容器内的元素会溢出：

```
<style> 
.flex-container {
    display: -webkit-flex;
    display: flex;
    width: 400px;
    height: 250px;
    background-color: lightgrey;
}

.flex-item {
    background-color: cornflowerblue;
    width: 200px;
    height: 100px;
    margin: 10px;
}
</style>
<div class="flex-container">
  <div class="flex-item">flexitem1</div>
  <div class="flex-item">flexitem2</div>
  <div class="flex-item">flexitem3</div>
  <div class="flex-item">flexitem4</div>
  <div class="flex-item">flexitem5</div>
</div>
```

![img](/img/luwencai/flex2.png)

### 5，flexbox属性

| 属性                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [display](https://www.w3cschool.cn/cssref/pr-class-display.html) | 指定 HTML 元素盒子类型。                                     |
| [flex-direction](https://www.w3cschool.cn/cssref/css3-pr-flex-direction.html) | 指定了弹性容器中子元素的排列方式                             |
| [justify-content](https://www.w3cschool.cn/cssref/css3-pr-justify-content.html) | 设置弹性盒子元素在主轴（横轴）方向上的对齐方式。             |
| [align-items](https://www.w3cschool.cn/cssref/css3-pr-align-items.html) | 设置弹性盒子元素在侧轴（纵轴）方向上的对齐方式。             |
| [flex-wrap](https://www.w3cschool.cn/cssref/css3-pr-flex-wrap.html) | 设置弹性盒子的子元素超出父容器时是否换行。                   |
| [align-content](https://www.w3cschool.cn/cssref/css3-pr-align-content.html) | 修改 flex-wrap 属性的行为，类似align-items, 但不是设置子元素对齐，而是设置行对齐 |
| [flex-flow](https://www.w3cschool.cn/cssref/css3-pr-flex-flow.html) | flex-direction 和 flex-wrap 的简写                           |
| [order](https://www.w3cschool.cn/cssref/css3-pr-order.html)  | 设置弹性盒子的子元素排列顺序。                               |
| [align-self](https://www.w3cschool.cn/cssref/css3-pr-align-self.html) | 在弹性子元素上使用。覆盖容器的 align-items 属性。            |
| [flex](https://www.w3cschool.cn/cssref/css3-pr-flex.html)    | 设置弹性盒子的子元素如何分配空间。                           |

flexbox属性介绍详情参考:[w3cschool](https://www.w3cschool.cn/css3/2h6g5xoy.html)

在这里重点介绍一下flex属性：

首先明确一点是， `flex` 是 `flex-grow`、`flex-shrink`、`flex-basis`的缩写。各个值解析:

- none：none关键字的计算值为: 0 0 auto
- [ flex-grow ]：定义弹性盒子元素的扩展比率。
- [ flex-shrink ]：定义弹性盒子元素的收缩比率。
- [ flex-basis ]：定义弹性盒子元素的默认基准值。

`flex` 的默认值是以上三个属性值的组合。假设以上三个属性同样取默认值，则 `flex` 的默认值是 0 1 auto。

当 `flex` 取值为 `none`，则计算值为 0 0 auto，当 `flex` 取值为 `auto`，则计算值为 1 1 auto。

当 `flex` 取值为一个非负数字，则该数字为 `flex-grow` 值，`flex-shrink` 取 1，`flex-basis` 取 0%。

当 `flex` 取值为一个长度或百分比，则视为 `flex-basis` 值，`flex-grow` 取 1，`flex-shrink` 取 1，（注意 0% 是一个百分比而不是一个非负数字）。

当 `flex` 取值为两个非负数字，则分别视为 `flex-grow` 和 `flex-shrink` 的值，`flex-basis` 取 0%。

当 `flex` 取值为一个非负数字和一个长度或百分比，则分别视为 `flex-grow` 和 `flex-basis` 的值，`flex-shrink` 取 1。