---
title: css那些我踩过的坑-1
date: 2018-07-05 10:40:34
tags: [css3] 
categories: 甘潇敏 
description: 记录做网页时出现的问题和解决方法
photos: 
     "http://image5.tuku.cn/pic/wallpaper/youxidongman/longmaokatongbizhi/001.jpg"
---

转载请申明出处

# css那些我踩过的坑-1

最近在把我去年写的一个网页扩展并升级，也解决了原来的一些问题，当时一些bug觉得怎么都改不了，烦的要死。结果前段时间看了一些别的代码，才发现我对一些标签的认知出现了错误，当时还一根筋的以为没错，才会改来改去都没效果，南辕北辙。所以为了以后不再犯这种错误，我想把我的一些踩坑经验写在博客上，既是对自己的一个警醒，又可以加深认识。

### z-index

z-index属性设置的是元素的堆叠顺序，w3c教程里面说的是这样的：拥有更高堆叠顺序的元素总是会处于堆叠顺序较低的元素的前面。 这解释真是太不亲民了，什么叫更高堆叠顺序？数字大的就是更高堆叠顺序吗？前面又是指什么？上面还是下面？

我当时做的网页需要把弹窗设置在轮播广告图的上面，就是鼠标悬停在导航上的时候，出现的弹窗会遮住轮播图，但是当时设置z-index的时候把这句话理解错误，还以为z-index越小，元素越前面，结果怎么改都没有效果，前端时间终于想明白了，才发现z-index属性设置越大，堆叠的越上面，例如：

```css
/*轮播图设置堆叠顺序为0*/
#slider{
    width:780px;
    height:350px;
    position:relative;
    overflow:hidden;
    z-index:0;
}
/*弹窗设置堆叠顺序为3*/
#main-left>li:hover div.surronding{
    display:inline-block;
    z-index:3;
    position:absolute;
}
/*#main-left>li:hover div.surronding在#slider上面*/
```

这么设置之后，鼠标悬停时出现的弹窗才会在轮播图上面，也就是我们看网页的时候先看到div元素，去掉div元素才能看到id=slider的元素。

**注意：z-index仅能在定位元素上奏效，例如设置了position的元素。**

### 去掉按钮点击之后出现的蓝色轮廓

前段时间做的网页里面一个按钮点击之后按钮会出现蓝色的边框，如图

![](/img/button1.png)

要去掉这一圈蓝色的边框，需要设置一个属性outline

```css
button{outline:none;}
```

在w3c教程里是这样写的：轮廓（outline）是绘制于元素周围的一条线，位于边框边缘的外围，可起到突出元素的作用。CSS outline 属性规定元素轮廓的样式、颜色和宽度。

设置了outline属性为none之后，按钮点击后就不会出现蓝色的边框了。

### 鼠标悬停时出现阴影

如图

![](/img/button2.png)

```css
#search-box button:hover{
    cursor:pointer;
    box-shadow:1px 1px 10px #DA4453 ;
}
```

设置box-shadow属性就可以出现这种阴影效果，下面的这段引用自w3c教程：

```css
box-shadow: h-shadow v-shadow blur spread color inset;
```

| 值         | 描述                                     | 测试                                                         |
| ---------- | ---------------------------------------- | ------------------------------------------------------------ |
| *h-shadow* | 必需。水平阴影的位置。允许负值。         | [测试](http://www.w3school.com.cn/tiy/c.asp?f=css_box-shadow) |
| *v-shadow* | 必需。垂直阴影的位置。允许负值。         | [测试](http://www.w3school.com.cn/tiy/c.asp?f=css_box-shadow) |
| *blur*     | 可选。模糊距离。                         | [测试](http://www.w3school.com.cn/tiy/c.asp?f=css_box-shadow&p=3) |
| *spread*   | 可选。阴影的尺寸。                       | [测试](http://www.w3school.com.cn/tiy/c.asp?f=css_box-shadow&p=7) |
| *color*    | 可选。阴影的颜色。请参阅 CSS 颜色值。    | [测试](http://www.w3school.com.cn/tiy/c.asp?f=css_box-shadow&p=10) |
| inset      | 可选。将外部阴影 (outset) 改为内部阴影。 |                                                              |

