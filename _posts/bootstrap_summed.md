---
title: bootstrap使用总结
date: 2018-05-26 22:05:00 
tags: [bootstrap] 
categories: 甘明阳
description: 记录bootstrap的一些常用内容使用
---

使用bootstrap框架开发可以极大的提高开发效率，bootstrap有以下优点：

1、响应式设计，bootstrap的设计无时不刻都是适应移动端的，我们不需要再特地写相应式了，提高效率

2、浏览器兼容，我以前开发都是引用bootstrap里面的normalize.css，基本上对绝大多数浏览器都能做到兼容

3、不需要自己取名字，说实话类名真的不好取，bootstrap里面把类名都给我们写好了，直接引用就可以了

由于bootstrap要记太多东西了，好记性不如烂笔头，我打算用这篇文章记录bootstrap的一些常用内容。方便开发的时候直接查阅

#### 引用bootstrap

[点击下载](https://pan.baidu.com/s/19CcqBNUXqGTqN8moieOo4Q)

我用的是版本是3.3.5。

#### 栅格系统

bootstrap将屏幕分为12等分，类名前面都co-为前缀,col的全称是column，中文翻译是列，例如想要4等分，每一份长度是屏幕宽度的三分之一就用col-md-3

```html
 <div class="col-md-3">
     Lorem ipsum dolor sit amet, consectetur adipisicing elit. 
 </div>
 <div class="col-md-3">
     Lorem ipsum dolor sit amet, consectetur adipisicing elit.
 </div>
 <div class="col-md-3">
     Lorem ipsum dolor sit amet, consectetur adipisicing elit. 
 </div>
 <div class="col-md-3">
     Lorem ipsum dolor sit amet, consectetur adipisicing elit.
 </div>
```

这个样式是响应式布局的，在手机上自动变为摞起来的样式，假如想让响应式布局生效的范围更小，只需要改md就行

范围从大到小依次是：

```bash
col-lg-3   //lagre
col-md-3   //medium
col-sm-3   //small
col-xs-3   //extra small
```

用的最多的是md和sm，一般很少用xs，因为xs不管屏幕多小都不会应用相应式。还是感觉这个bootstrap的栅格系统非常强大，原来不用bootstrap的时候我要实现这个功能要写不少的代码，现在直接引用就可以了。

#### 表单

表单在网页上的应用是非常多的，基本上每个网站都有登录注册输密码的页面，这些页面都是用表单做的。下面是一些例子：

##### 常用表单：

```html
 <form class="container" style="max-width: 550px;">
       <h1>注册</h1>
       <div class="form-group">
           <label>姓名</label>
           <input type="text" class="form-control">
       </div>
       <div>
           <label>密码</label>
           <input type="password" class="form-control">
       </div>
 </form>
```

效果如图：

![Q图片2018052623114](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_01.png)

这里最重要的样式就是form-control，给input用的，form-group的只是是添加15px的margin-bottom。

##### 内联表单

```html
 <form class="container" style="max-width: 550px;">
       <div class="form-inline">
           <h1 style="margin-bottom:15px;">注册</h1>
           <div class="form-group">
               <label>姓名</label>
               <input type="text" class="form-control">
           </div>
           <div class="form-group">
               <label>密码</label>
               <input type="text" class="form-control">
           </div>
        </div>
   </form>
```

效果如图：

![Q图片2018052623413](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_02.png)

这个样式只是多嵌套了一层form-inline的div，并且在手机上面和上面显示一样，感觉更加具有通用性

##### 验证结果表单

```html
<form class="container" style="max-width: 550px;">
       <div class="form-inline">
           <h1 style="margin-bottom:15px;">注册</h1>
           <div class="form-group has-error">
               <label class="control-label">姓名</label>
               <input type="text" class="form-control">
           </div>
           <div class="form-group">
               <label>密码</label>
               <input type="text" class="form-control">
           </div>
        </div>
    </form>
```

效果如图：

![Q图片2018052623500](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_03.png)



另外bootstrap还提供了其他验证结果的样式，比方说验证成功是has-success，验证警告是has-warning。

##### 充值金额表单

```html
  <form class="container" style="max-width: 550px;">
        <div class="form-group">
            <label style="margin-top: 15px;">充值金额</label>
            <div class="input-group">
                <div class="input-group-addon">￥</div>
                <input type="text" class="form-control">
            </div>
        </div>
  </form>
```

效果如图：

![Q图片2018052700015](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_04.png)

##### 密钥表单

```html
 <form class="container" style="max-width: 550px;">
       <div class="row" style="margin-top: 15px;">
           <div class="col-sm-3">
               <input type="text" class="form-control" placeholder="XXXX">
           </div>
           <div class="col-sm-3">
               <input type="text" class="form-control" placeholder="XXXX">
           </div>
           <div class="col-sm-3">
               <input type="text" class="form-control" placeholder="XXXX">
           </div>
           <div class="col-sm-3">
               <input type="text" class="form-control" placeholder="XXXX">
           </div>
       </div>
 </form>
```

效果如图：

![Q图片2018052700142](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_05.png)

这种表单结合了通用表单和栅格系统，其实和上面差不多。

#### 按钮

##### 一般按钮

相比表单，按钮的应用更广，几乎所有页面都有按钮，按钮的处理也相对简单，bootstrap提供了5种情况的按钮，分别是

```bash
<button class="btn btn-default">你好</button>  //默认按钮
<button class="btn btn-primary">你好</button>  //主色调按钮
<button class="btn btn-danger">你好</button>   //危险按钮
<button class="btn btn-warning">你好</button>  //警告按钮
<button class="btn btn-info">你好</button>     //信息按钮
```

代码：

```html
<form class="container" style="max-width: 550px;">
       <div style="margin-top:15px;">
           <button class="btn btn-default">你好</button>
           <button class="btn btn-primary">你好</button>
           <button class="btn btn-danger">你好</button>
           <button class="btn btn-warning">你好</button>
           <button class="btn btn-info">你好</button>
       </div>
</form>
```

效果如下：

![Q图片2018052700270](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_06.png)



其他标签像a标签和input的submit标签也可以使用这些样式，大小也是可以调整的。

```html
<form class="container" style="max-width: 550px;">
       <div class="margin-t15">
           <button class="btn btn-default btn-lg">你好</button>
           <button class="btn btn-primary">你好</button>
           <button class="btn btn-danger">你好</button>
           <button class="btn btn-warning btn-sm">你好</button>
           <button class="btn btn-info btn-xs">你好</button>
           <button class="btn btn-block btn-danger" style="margin-top: 15px;">你好</button>
       </div>
 </form>
```

效果如图：

![Q图片2018052700352](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_07.png)

在手机银行的项目上btn-block用的最多。

##### 按钮组

在很多后台项目像什么管理平台会用到按钮组，这里也写一下：

```html
<form class="container" style="max-width: 550px;">
       <!--按钮组-->
       <div class="margin-t15">
           <div class="btn-toolbar">
               <!--按钮组是btn-group-->
               <div class="btn-group btn-group-lg">
                   <button class="btn btn-default">你好</button>
                   <button class="btn btn-default">你好</button>
                   <button class="btn btn-default">你好</button>
               </div>
               <div class="btn-group">
                   <button class="btn btn-default">你好</button>
                   <button class="btn btn-default">你好</button>
                   <button class="btn btn-default">你好</button>
               </div>
               <div class="btn-group btn-group-sm">
                   <button class="btn btn-default">你好</button>
                   <button class="btn btn-default">你好</button>
                   <button class="btn btn-default">你好</button>
               </div>
           </div>
       </div>
       <div style="margin-top:15px;">
            <div class="btn-group-vertical">
                <button class="btn btn-default">你好</button>
                <button class="btn btn-default">你好</button>
                <button class="btn btn-default">你好</button>
            </div>
       </div>
   </form>
```

效果如图：

![Q图片2018052700415](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_08.png)

下面默认添加样式

```css
body{
    max-width: 400px;
    margin: 20px auto;
}
```



#### 导航

关于导航栏，主要有下面4种：

##### 知乎导航

```html
<ul class="nav nav-tabs">
    <li class="active"><a href="#">登录</a></li>
    <li><a href="#">注册</a></li>
    <li><a href="#">忘记密码</a></li>
</ul>
<div>
    <h1>登录</h1>
    Lorem ipsum dolor sit amet, consectetur adipisicing elit. Voluptates tempore minus similique doloribus, at error odio laudantium ad blanditiis neque inventore pariatur voluptate quis exercitationem? Nulla cumque labore magnam dolor.
</div>
```

效果如下：

![Q图片2018060220210](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_09.png)

这里说明一下，在nav-tabs后面加上nav-justified的类可以是导航按钮变得更宽

```html
<ul class="nav nav-tabs nav-justified">
    <li class="active"><a href="#">登录</a></li>
    <li><a href="#">注册</a></li>
    <li><a href="#">忘记密码</a></li>
</ul>
<div>
    <h1>登录</h1>
    Lorem ipsum dolor sit amet, consectetur adipisicing elit. Voluptates tempore minus similique doloribus, at error odio laudantium ad blanditiis neque inventore pariatur voluptate quis exercitationem? Nulla cumque labore magnam dolor.
</div>
```

效果如下：

![Q图片2018060220220](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_10.png)

##### 普通导航

```html
<ul class="nav nav-pills nav-pills">
    <li class="active"><a href="#">登录</a></li>
    <li><a href="#">注册</a></li>
    <li><a href="#">忘记密码</a></li>
</ul>
<div>
    <h1>登录</h1>
    Lorem ipsum dolor sit amet, consectetur adipisicing elit. Voluptates tempore minus similique doloribus, at error odio laudantium ad blanditiis neque inventore pariatur voluptate quis exercitationem? Nulla cumque labore magnam dolor.
</div>
```

效果如下：

![Q图片2018060220222](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_11.png)

##### 侧边栏导航

```html
<div class="row">
    <div class="col-xs-4">
        <ul class="nav nav-pills nav-stacked">
          <li class="active"><a href="#">登录</a></li>
          <li><a href="#">注册</a></li>
          <li><a href="#">忘记密码</a></li>
        </ul>
    </div>
    <div class="col-xs-8">
        <div>
            <h1>登录</h1>
            Lorem ipsum dolor sit amet, consectetur adipisicing elit. Voluptates tempore minus similique doloribus, at error odio laudantium ad blanditiis neque inventore pariatur voluptate quis exercitationem? Nulla cumque labore magnam dolor.
        </div>
    </div>
</div>
```

效果如下：

![Q图片2018060220223](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_12.png)

其实这个只是普通导航和栅格系统的组合使用

#### 导航栏

```html
<div class="navbar navbar-default">
    <div class="navbar-header">
        <div class="navbar-brand">名言网</div>
    </div>
    <ul class="nav navbar-nav">
        <li><a href="#">首页</a></li>
        <li><a href="#">产品</a></li>
        <li><a href="#">联系我们</a></li>
    </ul>
    <form class="navbar-form navbar-left">
        <div class="form-group">
            <input type="text" class="form-control">
        </div>
        <button class="btn btn-danger" type="submit">搜索</button>
    </form>
     <ul class="nav navbar-nav navbar-right">
        <li><a href="#">登录</a></li>
        <li><a href="#">注册</a></li>
    </ul>
</div>
```

效果如图：

![Q图片2018060220241](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_13.png)

由于导航栏需要占据屏幕整宽，这里没有限制body宽度，在navbar里面使用navbar-left或者navbar-right实现左对齐或者右对齐

#### 面板

```html
<div class="panel panel-default">
    <div class="panel-heading">
       <div class="panel-title">用户统计</div>
    </div>
    <div class="panel-body">
        Lorem ipsum dolor sit amet, consectetur adipisicing elit. Blanditiis aperiam nostrum, hic voluptas, consequatur magni libero fugiat architecto at, excepturi quis distinctio, laboriosam eos. Ipsam voluptates maiores possimus magnam animi.
    </div>
    <div class="panel-footer">
        <div class="small text-muted">数据更新于5秒前</div>
    </div>
</div>
```

效果如下：

![Q图片2018060220244](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_14.png)

这里text-muted和small主要是将字号变小并且改变颜色的，并且panel-default可以改成panel-success或者panel-warning

#### 表格

和表单不同，表格适用于统计信息的

```html
<table class="table table-striped">
        <thead>
            <tr>
                <th>用户名</th>
                <th>联系方式</th>
                <th>注册于</th>
            </tr>
        </thead>
        <tbody>
            <tr class="danger">
                <td>wanghuahua</td>
                <td>wanghuanghuan1@com</td>
                <td>2048年</td>
            </tr>
            <tr class="success">
                <td>你阿飞</td>
                <td>wanghuanghuan1@com</td>
                <td>2048年</td>
            </tr>
            <tr class="warning">
                <td>235432</td>
                <td>wanghuanghuan1@com</td>
                <td>2048年</td>
            </tr>
             <tr class="info">
                <td>sgsg</td>
                <td>wanghuanghuan1@com</td>
                <td>2048年</td>
            </tr>
        </tbody>
    </table>
```

效果如下：

![Q图片2018060220253](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_15.png)



table-striped是用来加重基数行的颜色的，这里由于全部加上了颜色没有表现出来，可以改为table-hover用来相应鼠标滑动效果，想要边框可以在table里面添加类table-bordered

#### 翻页

```html
<ul class="pagination">
   <li><a href="#">上一页</a></li>
   <li class="active"><a href="#">1</a></li>
   <li><a href="#">2</a></li>
   <li><a href="#">3</a></li>
   <li><a href="#">...</a></li>
   <li><a href="#">50</a></li>
   <li><a href="#">下一页</a></li>
</ul>
```

效果如下

![Q图片2018060220271](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_16.png)

还有一种简单的翻页效果，主要是用于页数比较少的

```html
 <ul class="pager">
   <li><a href="#">上一页</a></li>
   <li class="disabled"><a href="#">下一页</a></li>
</ul>
```

效果如下：

![Q图片2018060220272](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_17.png)

#### 文章头部

```html
 <div class="breadcrumb">
   <li><a href="#">首页</a></li>
   <li><a href="#">文章列表</a></li>
   <li class="active">yo. world <span class="badge">15k</span></li>
</div>
```

效果如下：

![Q图片2018060221384](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_18.png)

#### 列表

```html
 <div class="list-group">
   <a href="#" class="list-group-item">Item</a>
   <a href="#" class="list-group-item">Item</a>
   <a href="#" class="list-group-item">Item</a>
   <a href="#" class="list-group-item">Item</a>
</div>
```



效果如下：

![Q图片2018060221415](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_19.png)

这种列表主要是用作侧边栏的

#### 标签

```html
<p>
   <span class="label label-success">面霜</span>
   <span class="label label-info">water baby</span>
   <span class="label label-danger">http</span>
</p>
```

效果如下：

![Q图片2018060221442](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/bootstrap_20.png)

这种标签可以完美的嵌入文字中间