---
title: js函数深入研究
date: 2018-07-16 21:54:00 
tags: [js] 
categories: 甘明阳
description: 记录js的进阶使用
---

javascript入门容易精通难，由于语法比较松散，很多用法需要我们自己去研究，这段时间我主要是研究js函数的使用

##### 使用函数作为参数

在js里面函数也是一种数据类型，我们可以使用函数作为另外一个函数的参数，例如：

```javas
//定于被调用的函数
function hello(){
    for(var i=0;i<5;i++){
        console.log(i);
    }
}
//定义去调用的函数
function go(fn){
    window[fn].call(this);
}
//调用，这里直接写hello就可以啦，后面不用小括号哦
go("hello");
```

##### 使用函数(带参)作为参数

感觉上述方法在实际效果中并不好用，因为实际情况一般都有参数，下面来介绍一个带参数函数的使用

```javas
//定义主动调用函数
function test1(value,Func){
        Func(value)
}
//定义被调用函数
function test3(data)
{
    alert(data);
}
//去调用
test1('22',test3);
```

在test1里面有两个参数，被调用函数的参数，和调用函数，

##### ajax封装

将上面的知识引申到实际代码里面，比如说ajax方法的封装

下面是一个ajax的基本使用：

```javas
$.ajax({
        //这是请求的servlet
        url: "hqyzm",
        //这里定义请求类型
        method: "post",
        //这里是前端要传给后台的数据
        data: {
            phone: "12345678912"
        },
        //这里是请求成功调用的方法
        success: function(data){
            console.log(data)
        },
        //这里是请求失败调用的方法
        error: function(data){
            alert("网络繁忙，请稍后重试。");
        }
    })
```

项目会大量的调用ajax，如果每次我们都去单独写ajax会让项目代码非常臃肿，我们应该将ajax封装起来，可以根据上面的知识来封装ajax

###### 思路分析

ajax主要有三个地方会变化：1、请求的servlet          2、传给servlet的参数        3、请求成功调用的方法。

我们可以用变量代替这些数据，下面是一个注册查询数据库是否已存在用户名的ajax请求

```javasc
//封装一个ajax方法，这里execute是请求成功执行的方法，servlet是去请求的servlet，val传过去的参数
function to_ajax(execute,servlet,val){
    $.ajax({
        url: servlet,
        method: "post",
        data: {
            username: val,
        },
        success: function(data){
            execute(data)
        },
        error: function(){
            alert("网络繁忙，请稍后重试。")
        }
    })
}
//定义ajax请求成功执行的方法
function check_name_to_db(data){
   if(data=="0"){
        $("#name_detail").html("该称呢已被注册");
        $("#title_name").css("display","block");
        name_status = 0;
   }
}
//调用这个ajax方法
to_ajax(check_name_to_db,"Checkname","zhangsan");

```

这个函数还可以进一步封装，主要在于参数的传送，毕竟每次ajax传送的参数数量都不一定相同，要解决这个问题需要运用java面向对象的思维，将参数作为一个对象传送给服务器端，例如上传文件的时候将参数封装到FormData对象里面

```javascript
 var formdata = new FormData();
 //想formdata里面添加数据
 formdata.append("file",file);
 //调用ajax方法
 up_ajax(req_success,"Mybook",formdata)
```

##### 参数的封装

js函数的参数都是封装到一个名为arguments的数组里面，里面按照从前到后顺序存放参数，例如：

```javas
//定义被调用的函数
function say(){
      console.log('Hello' + arguments[0] + arguments[1] + arguments[2]);
      console.log(arguments.length);

}
//调用函数
say("World！", "ByeBye!","gogo!!");
```

控制台打印结果是HelloWorld！ByeBye!gogo!!，由此可见，我们使用函数的时候可以写10个8个实参，但是定义的时候

不写形参也可以。

最终封装结果，封装了一个上传组件

```javascript
//下面是上传文件使用的ajax方法
function up_ajax(execute,servlet,val_data){
    $.ajax({
        url: servlet,
        method: "post",
        data: val_data,
        //将datatype改为text放在上传成功却进入error方法
        dataType : "text",
        processData: false,  // 注意：让jQuery不要处理发送的数据
        contentType: false,  // 注意：让jQuery不要设置contentType请求头
        success: function(data){
            execute(data);
        },
        error:function(){
            alert("网络繁忙，请稍后重试！")
        }
    })
}

function upload_book(){
    //$("#upload_book").prop("files")是获取files属性的值，返回的结果是一个数组
    //这个数组里面有两个键值对 0:file,length:1，通过key为0获取file文件
    //file是一个file对象，里面封装了上传的文件的各种属性
    var file =  $("#upload_book").prop("files")["0"];
    //获取文件的大小，将单位转为m
    var size = file.size/1024/1024;
    //如果上传文件的大小超过3M就不让上传
    if(size>3){
        alert("上传文件过大");
    }else if(size<=0){
        alert("不能上传空文件");
    }else{
       //由于上传的是二进制文件，需要创建FormData对象
        var formdata = new FormData();
        //想formdata里面添加数据
        formdata.append("file",file);
        //调用ajax方法
        up_ajax(req_success,"Mybook",formdata)
    }
}
//上传成功调用
function req_success(){
    var data = arguments[0];
    console.log(data);
}


```



