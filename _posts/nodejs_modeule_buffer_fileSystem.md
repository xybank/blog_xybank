---
title: nodejs-module-buffer-fileSystem
date: 2018-05-04
tags: [nodejs]
categories: 童长钱
description: module、buffer、fileSystem相关知识
---
## 一、MODULE
#### 1.1、 JavaScript的模块规范  
目前流行的JavaScript的模块规范主要有两种：  
**CommonJS**  
*CommonJS*把每一个文件看做一个模块，模块内部定义的每一个变量都是私有的，无法别其他模块使用，除非使用预定义的方法将内部定义的变量暴露出来（*通过exprots和require关键字来实现*）  
*CommonJS* 一个显著的特点就是模块加载是同步的，就目前来说，受限制于宽带速度，并不适用于浏览器中的人JavaScript。  
**AMD**  
*AMD*是Asynchronous Module Definition的缩写，意思就是“一步模块定义”，它采用异步方式加载模块，模块加载不影响它后面语句的运行。依赖这个模块代码定义在一个回调函数中，等到加载完成之后，这个回调才会运行。  
<!--more-->
#### 1.2、require及其运行机制  

``` node.js
//preson.js
var preson ={
    talk: function () {
        console.log("I am talking..."); 
    },
    listen :function () {
        console.log("I am listening..."); 
    }
    //MORE func...
}
module.exports = person;
```  

这样实现一个自定义的模块，该模提供的一个preson接口，然后使用module.exports将接口口暴露给外部有使用，外部的代码想要使用preson.js的方法，需要使用require关键字引入该接口。  

```node.js
var persom =requitr("./person.js");
person.talk();
```  

**注意**：再引入自定义模块时省略相对路径“./”会导致错误。  
如果只需要引入其中的人一个，代码可以这样写：  

```node.js
var talk =require("./person.js").talk;
talk();
```  

**require**关键字并不依赖于exports，应为加载一个没有暴露任何方法的模块，这相当于执行一个模块代码，这样做通常是没有意义的。   

#### 1.3、require的缓存策略  

Node会自动缓存require引入的文件，使得下次再引入不需要在经过文件系统而是直接从缓存读取。这种缓存是基于文件路径定位的，这表示两个完全相同的文件，但在他们位于不同的路径下，也会在缓存中维持两份。例如使用下面代码访问目前在缓存中的文件：  

```node.js
var moudle =require("./moudle.js");
console.log(require.cache);
```

#### 1.4、require的隐患
当调用require加载一个模块时，模块内部的代码会被调用，有时候会带来隐藏的bug。例如下面的代码：  

```node.js
//module.js
function test(){
    setInterval(function (){
        console.log("test");
    }),100
}
test();
module.expoets =test;
//run.js
var test require("./module.js");
```
run.js除了加载一个模块之外没有进行任何操作，试着运行一下，就会发现每隔一秒钟就会输出一个 “test”字符串，同时run.js的进程不会退出。因为：加载一个模块相当于执行模块内部代码，在module.js中由于设置了一个不间断的定时器，导致run.js也会一直运行。 

#### 1.5、模块化下的作用域
主要关注点在**this**上。  
Node和javaScript中的this指向上有一些区别，其中Node控制台和脚本文件的策略不尽相同。
对于浏览器中的javaScript来说，无论是在脚本还是在浏览器控制台中，其this的指向是一致的，然Node并非这样。
1. 控制台中的this

``` nodejs
console.log(this);
//输出：window{stop:functiom,open function,.....}
//---------------------
var a = 10;
console.log(this.a);
//输出 10;
```
2. Node控制台中的this
在node控制台中,全局变量会挂到global下.  
创建一个test.js文件在文件中添加以下代码：
``` nodejs
console.log(this);//{}
```
运行 node test.js打印出结果是一个空对象。
执行下面代码：
``` nodejs
var a = 10;
console.log(this.a);//undefined
console.log(global.a);//undefined
```
结果全部都是undefined,说明定义的变量a并没有挂到全局变量的this或者global对象。  
然而如果声明变量时不使用var或者let关键字修饰，例如下面代码：
``` nodejs
a = 10;
console.log(global.a);//10
```
这样却可以正常打印结果。
node脚本文件中定义的全局this指向是：module.exports。如下面代码所示：

``` nodejs
this.a = 10;
console.log(module.exports);//10
```
#### 1.6、Node中作用域种类
1. 全局作用域
``` nodejs
a = 10;
console.log(global.a);//10
```
2. 模块作用域
``` nodejs
this.a = 10;
console.log(module.exports);//10
```
3. 函数作用域  
这个大家都比较熟悉，就不在介绍。  

4. 块级作用域  
ES2015中引入let关键字提供了对块级作用域的支持。

## 二、Buffer  
#### Buffer？  
**Buffer**是Node特有的一种数据类型，主要用来处理二进制数据。  

**Buffer**属于固有（built）类型无需使用require引入。  

例如下面例子：    
``` node.js
var fs = require("fs");
fs.readFile("foo.text",function (err,results){
    console.log(results);
});
//结果：<Buffer 48 65 6c 6f 20 57 6f 72 6c 64 >(hellow node)  
```    
上面的代码中，最后打印出的是十六进制的数据，由于纯二进制格式太长而且难以阅读，Buffer通常表现为十六进制的字符串。  


#### Buffer的构建以转换  

buffer类初始化一个buffer对象，参数可以是二进制数据组成的数组：    
``` node.js
var buffer = Buffer([0x48,0x65,0x6c,0x6c,0x6c,0x6f,0x20,0x4e,0x64f,0x64,0x65]);  
//结果：hellow node  
```  
如果想有一个字符串来得到一个Buffer，同样可以实现，例如：  
``` node.js
var buffer = new Buffer("Hellow Node");
console.log(buffer);
//结果：<Buffer 48 65 6c 6f 20 4e 6f 64 65>
```  
**注意**：在最新的Node API中，Buffer()方法被标记为Depecated(“不赞同”的意思)，表示不推荐使用;目前推荐使用Buffer.from方法来初始化一个对象，上面的代码可以改写成为提下形式：   
``` node.js
var buffer = Buffer.from([0x48,0x65,0x6c,0x6c,0x6c,0x6f,0x20,0x4e,0x64f,0x64,0x65]);//"Hellow Node"
var buffer = new Buffer.from("Hellow Node");
console.log(buffer);
//结果：<Buffer 48 65 6c 6f 20 4e 6f 64 65>
```   
如果想把一个Buffer对象转换成字符串的形式，需要用到toString()方法，调用格式为：  
``` node.js
buffer。toString([encoding],[start],[end]);
encoding:目标编码格式
start:起始位置
end:结束位置
```   
Bufer支持的编码格式有限，只有以下六种：
1. ASCⅡ
2. Base64
3. Binary
4. hex
5. UTF-8
6. UTF-16LE/UCS-2  

buffer还提供了isEncoding方法来判断是否支持转换为目标编码格式。   

例如：把“Hellow Node”的Buffer对象转换为字符串，可以这样调用：    
``` node.js
//制转换前五个字符 输出“hello”
console.log(buffer.tiString("utf-8",0,5));
```   
#### Buffer 的拼接

Buffer一个常见的场景是用来处理HTTP的post请求，例如下面代码: 

``` node.js
var body = '';
req.setEncoding("utf-8");
req.on("data",function(chunk){
    body += chunk;
});
req.on("end",fiunction(){
});
```   

*上面代码使用+=来拼接上传的数据流，这个过程包含了一个隐式的编码转换。  
body += chunk 相当于 body+=chunk。toString()，当上传的字符全是英文字符的时候固然没有问题，  但如果包含中文或者其他语言，由于toString方法默认使用UTF-8编码，这个时候就可能出现乱码。*

**举个例子**：首先构造一个中文字符串，并保存为test.txt      
*八百标兵奔北坡，北坡炮兵并排跑。炮兵排把标兵碰，标兵怕碰炮兵炮。*
``` node.js
var rs =require("fs").createReadStream("test.txt",{highWaterMark :10});
var data = "";
rs.on("data",function (chunk){
    data += chunk;
});
rs.on("end",function(){
    console.log(data);
});
```   

**highWaterMark**    
字面的意思最高水位线,它表示内部缓冲区最多能容纳的字节数，如果超过这个大小，就停止读取资源文件，默认大小64KB。  
假设文件的大小超过默认值64KB就会触发data事件：chunk的大小就等于*highWaterMark*大小;直到文件读取完成时，触发end事件。    
由于UTF-8中一个汉字占三个字节，那么highWaterMark设置为10，每三个子之后就有一个字会被截段，因此在调用tostring方法的时候出现乱码。  
目前上面的代码已经被舍弃，官方推荐使用push方法来拼接Buffer，如下面代码所示：   

``` node.js
var rs =require("fs").createReadStream("test.txt",{highWaterMark :10});
var data = [];
rs.on("data",function (chunk){
    data.push(chunk);
});
rs.on("end",function(){
    var buf = Buffer.concat(data);
    console.log(buf.toString());
});
```  
上面代码在拼接的过程中不会有隐式的编码转换，首先将Buffer放到数组里面去，等待传输完后在进行转换，这样就不会出现乱码。
## 三、File System
#### File System？  
**File System** 是Node中使用最为频繁的模块之一，该模块提供了读写文件系统的能力。 
浏览器中的JavaScript没有读写本地文件系统的能力(忽略IE中的ActiveX),而Node作服务器编程语言，伟岸系统API是必须的，File System模块包含了数十个文件操作的API，[查阅API](https://nodejs.org/dist/latest-v8.x/docs/api//fs.html)
<!--more-->
#### 下面介绍几个常用的API
1. **readFile**

声明如下：    
``` node.js
fs.readFile(file[,options,callback])#  
Added: v0.1.29  
file <String> | <Buffer> | <Integer> filename of file descriptor  
options <Object> | <String>  
encoding <String> | <Null> default  = null  
flag <String> defult = "r"  
callblack = <Function>
```  
readFile 方法用异步来读取文件中的内容，例如：  

``` node.js
var fs= require("fs");
var data = fs.readFileSync("foo.txt",function (err,data){
    if (err) thorw err;
    console.log(data);
});
``` 
*readFile*会将一个文件的全部内容读到内存中，适用于提交较小的文本；如果有一个数百MB大小的文件需要读取，建议不要使用readFile而是选择stream。readFile读出的数据需要在回调方法中获取，而readFileSync直接返回文本数据内容。  

``` node.js
var fs= require("fs");
var data = fs.readFileSync("foo.txt",{encoding:"UTF-8"});
console.log(data);
```   

如果不指定File的encoding配置，readFile会直接返回类似Buffer格式；如果希望得到的是字符串形式，还需要调用toString方法进行转化.

2. **writeFile**  

声明如下：  

``` node.js
fs.writeFile(file, data[, option],callback);
file <String> | <Buffer> | <Integer> filename of file descriptor  
data <String> | <Bufer>
options <Object> | <String>  
encoding <String> | <Null> default  = null  
mode <Integer> defult = 0o666  
flag <String> defult = "w"  
callblack = <Function>
```   
在WriteFile的第一个参数文件名，如果不存在，则会尝试创建它(默认flag为w)。  

``` node.js
var fs = require("fs");
fs.writeFile("foo.txt", "你好", { flag: "a", encoding: "UTF-8" }, function (err) {
    if (err) {
        console.log(err);
        return;
    }
    console.log("success");
});
```  
3. **fs.stat(path,callback)**  
stat方法通常用来获取3文件状态。  
通常开发者可以再调用opne(),read(),或者write方法之前调用fs.stat方法，用来判断文件是否存在。

``` node.js
var fs = require("fs");
fs.stat("test.txt",function(err,result){
    if (err){
        console.log(err);
        return;
    }
    console.log(result);
});
```   

文件存在返回文件信息，如果不存在则会出现Error ENOENT：no such file or director的错误。  

4. **fs.stat和fs.fstat的区别**

声明如下：  

``` node.js
fs.fstat(fd,callback)
```  

fs.stat和fs.fstat在功能上是等价的，唯一的区别是fstat第一个参数是文件的描述符，格式为Integer，因此fstat方法通常搭配open方法使用，因为open方法返回得到结果就是一个文件描述。  
 
``` node.js
var fs = require("fs");
fs.open("test.txt","a",function (err,fd){
    if (err){
        console.log();
        return;
    }
    console.log(fd);
    fs.fstat(fd,function(err,result){
        if (err){
            console.log(err);
            return;
        }
        console.log(result);
    });
});
```   

下面是一个例子————获取目录下所有的文件名，这是一个常见的需求，实现这个功能只需要fs.readFile以及fs.stat连个API，readdir用于获取目录下所有的问文件或者子目录，stat用来判断具体每条记录是文件还是子目录，如果是子目录，则递归调用整个方法。  

``` node.js
var fs = require("fs");
function getAllFileFromPath(path) {
    fs.readdir(path, function (err, res) {
        for (var subPath of res) {
            //这里使用同步方法而非异步
            var statObj = fs.statSync(path + "/" + subPath);
            if (statObj.isDirectory()) {//判断是否为目录
                console.log("Dir:", subPath);
                getAllFileFromPath(path + "/" + subPath)
            } else {
                console.log("File:", subPath);
            }
        }
    })
}
getAllFileFromPath(__dirname);
``` 