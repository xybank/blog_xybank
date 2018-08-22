---
title: 使用jquery做图形单选框
date: 2018-08-022 10:54:00 
tags: [html, radio, checked] 
categories: 甘明阳
description: 使用jquery改动单选框选中状态
---

今天看了一下开源中国的注册页面，发现他们的性别选择做的很好看，不是使用原生单选框而是使用图片做的，如图：

![Q图片2018082210402](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/radio_01.png)

作为对界面要求高的人，怎么能忍受自己的注册页面还使用原生的单选框呢？

我开始思考怎么做出这种效果，首先是图片的问题，这个使用截图工具截下来，一共4个图片，然后是显示逻辑，这个使用jquery控制显示和隐藏，改变display的值就可以了，样式实现很简单，然后就是传值的问题，怎么让servlet知道用户的选择呢，有两个方案

1、在js里面设置一个页面变量，sexChecked，值为1表示选中男士，值为2表示选中女士，然后在提交的时候把sexChecked传给servlet就可以了。

2、在html里面写一个隐藏的单选框，用户选中男士的时候让id为manRadio的单选框变为选中状态，选中女生的时候让id为

womanRadio的单选框变为选择状态

由于我是使用form表单提交不是使用ajax提交数据的，form表单提交的时候数据没有经过js这一块，感觉不太好改，还是第二个方案比较简单，不需要改动form表单的参数，我选择第二个方案，代码如下

html代码：

```html
<div class="form-group padding_tb5">
    <div class="inlineBlock" id="man">
         <img src="img/default/man.png" class="vertical_mid">
         <span class="sex_text">男士</span>
    </div>
    <div class="inlineBlock hider" id="manHide" onclick="checkSex('1')">
        <img src="img/default/noselect.png" class="vertical_mid">
        <span class="sex_text">男士</span>
    </div>
    <div class="womanDiv hider" id="woman">
        <img src="img/default/woman.png" class="vertical_mid">
        <span class="sex_text">女士</span>
    </div>
    <div class="womanDiv margin_l25" id="womanHide" onclick="checkSex('2')">
        <img src="img/default/wonoselect.png" class="vertical_mid">
        <span class="sex_text">女士</span>
    </div>
    <input type="radio" name="sex" value="1" id="manRadio" checked="checked">
    <input type="radio" name="sex" value="2" id="womanRadio">
</div>
```

js代码：

```javascript
//性别选择判断
function checkSex(id){
    if(id=="1"){
        $("#manHide").css("display","none");
        $("#man").css("display","inline-block");
        $("#womanHide").css("display","inline-block");
        $("#woman").css("display","none");
        $("#manRadio").attr("checked","checked");
    }else if(id=="2"){
        $("#womanHide").css("display","none");
        $("#man").css("display","none");
        $("#woman").css("display","inline-block");
        $("#manHide").css("display","inline-block");
        $("#womanRadio").attr("checked","checked");
    }
}
```

这里出现了一个很神奇的问题，这个单选框居然只能改动一次，一开始我以为是没有进入if判断，使用console.log()打印却能打印出来，jquery里面的id也没有写错，代码看起来完全没有问题，这下我有点懵逼了，在网上找来找去也没有找到类型的问题，难道是不能给checked的值使用checked吗？我改成了false和ture，还是一样只能改动一次，几个假设都被推翻，确实很头疼，这个时候我想了一下，为什么第一次可以改动，肯定是这两个radio有什么不一样的地方，好像第一个radio一开始是checked的状态的，我把这个属性去掉了，再测试可以改动两次了，果然radio的checked会影响jquery的改动，改变下一个radio的时候去掉上一个radio的checked属性不就可以了吗，我加了一句remove的代码，果然可以改动啦，html还有这么神奇的地方真是的，改动如下

```javascript
//性别选择判断
function checkSex(id){
    if(id=="1"){
        $("#manHide").css("display","none");
        $("#man").css("display","inline-block");
        $("#womanHide").css("display","inline-block");
        $("#woman").css("display","none");
        //改变下一个radio的时候要移除已经选中的radio
        $("#womanRadio").removeAttr("checked");
        $("#manRadio").attr("checked","checked");
    }else if(id=="2"){
        $("#womanHide").css("display","none");
        $("#man").css("display","none");
        $("#woman").css("display","inline-block");
        $("#manHide").css("display","inline-block");
        //改变下一个radio的时候要移除已经选中的radio
        $("#manRadio").removeAttr("checked");
        $("#womanRadio").attr("checked","checked");
    }
}
```

这样改动就可以了，功能成功实现，效果如下：

![Q图片2018082211143](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/radio_02.png)

总结：平时做项目的时候要多思考，遇到问题仔细分析，多看看别人的技术博客，这样才能进步得更快。[页面跳转传送门](http://www.ganmyds.cn/servletPro/html/register.jsp)