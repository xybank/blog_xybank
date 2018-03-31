---
title: Nodejs入门
date: 2018-03-28 14:10:23
tags: [Nodejs]
categories: 余声赞
description: Nodejs简单入门
---

## 1.写在前面  

由于工作需要前一段时间自学了`Nodejs`，刚好又学习了个人的搭建，所以萌生把`Nodejs`基础记录下。   

此文档还需要配合对应的练习来看，直接看文字描述会比较模糊。对应练习文档在最后有提供。  
右边是对应练习文档的链接：[练习文档下载](/download/webstormWorkFile.rar")。  

`注意：此文档是本人看视频学习的手记。`    

<!--more-->

## 2.Node.js概述  

### （1）[官网：https://nodejs.org](https://nodejs.org)。  

在官网我们可以下载最新的`Nodejs`，目前最新版本是：8.10.0 LTS（可以简单理解为稳定版本，部署生产项目使用此版本）和9.9.0 Current（最新版本，相对不是很稳定的版本，尝试一些新功能等都可以使用此版本），我们还可以查看对应的api文档。  

### （2）Node.js是什么  

 + Node.js是使用C/C++编写的基于Chrome V8的JS运行时环境。  
 + Node.js是基于Chrome V8开发的；  
 + 有V8的一些功能，同时还拓展了一些其他功能；  
 + 不像Chrome中的console有window、document等dom和bom的对象;    

### （3）Node.js是一门基于ECMAScript开发的服务器端语言（php等也是服务器语言），提供了（前端JS没有的）很多拓展的对象。    
 
 + 前端js：   
1.ES原生对象：String,Number,Boolean,Math,Date,Error,Function,object,Array,RegExp...  
2.BOM（浏览器模型）&DOM（文件对象模型）  
3.自定义对象  

 + Node.js：  
1.ES原生对象
2.Node.js内置对象（没有BOM（浏览器模型）&DOM（文件对象模型））
3.大量的第三方对象
4.自定义对象

**Node.js可以编写独立的服务器应用，无需借助于其他的web服务器**  

 + Node.js的意义：  
1.执行效率比PHP/JSP/JAVA要快  
2.用一种语言统一了前端和后台的开发  

**特点：**  

 + 1.单线程逻辑处理  
 + 2.非阻塞  
 + 3.异步I/O处理  
 + 4.事件驱动编程    

## 3. Node.js的两种运行方式  

 + 1.交互模式—用于测试  
读取用户的输入、执行运算、输出结果、继续下一个循环。如在命令行下的
 
+ 2.脚本模式—用于开发  
把要执行的所有js语句编写在一个独立的文件中，一次性的提交给nodejs处理。此文件可以没有后缀（默认就是js文件）  
执行方法：node xx/xxx/xxx.js

## 4.IDE--webstorm

编码工具，自动检测安装的node，编写好文件后，右击--Run xxx即可	  

查看JavaScript的版本：  
file--setting--Language & Framew--JavaScript

## 5. Node.js的基础语法及ES6新特性  

### 1.数据类型  
 + （1）原始类型  
string,number,boolean,null...
原始类型数据直接赋值即可  

 + （2）引用类型  
ES原生对象、Node.js内置对象、自定义对象等
引用类型通常需要使用new关键字创建  

### 2.模板字符串  

ES6中提供的一种新的字符串形式     
（1）使用模板字符串定义字符串，数据可以实现换行  
（2）可以使用${}拼接变量，并且可以执行运算  

### 3.严格模式  

ES5中新增一种比普通模式更为严格的js运算模式  
使用方法；  
（1）在整个脚本文件中启用严格模式  
在脚本文件的开头：`"use strict"`;  
用于新项目  

（2）在单个函数内启动严格模式  
```
function info(){
	"use strict";
	...
}
```
用于老项目维护等  

规则：  

 + （1）常量不可修改，修改是非法的--将静默失败升级为错误  
node.js版本不一样执行结果不一样  
+ （2）不允许对未声明的变量赋值    
+ （3）匿名函数的this不再指向全局  

### 4.变量的作用域  

全局作用域、局部作用域、块级作用域（ES6中专有）  
块级作用域：只有在当前代码块中使用  
代码块：任何一个{}都是一个代码块，if/for/while...  
代码块中使用"let"声明块级作用域变量，出了代码块不再可用。使用let需要启用严格模式。  

### 5.循环结构

（1）for...of...(ES6)
遍历数组的元素值

### 6.函数

匿名函数的自调;

=>箭头函数  
```
(()=>{

})();
```
箭头函数只用于匿名函数  
箭头函数中不存在arguments对象

### 7.对象

创建对象的方式：  

 - （1）对象直接量  
 - （2）构造函数方式  
 - （3）原型继承方式  
 - （4）class方式--ES6新增的  

class，类：是一组相似对象的属性和行为的抽象集合。（描述一类事物统一的属性和功能的程序结构）
事物的属性是class的属性，事物的功能是class的方法  
使用class方式创建自定义对象时，必须启用严格模式  
```
"use strict";
class Emp{
	constructor(ename,salary){//声明一个构造函数
	this.ename=ename;
	this.salary=salary;
}
work(){//定义一个方法,和constructor并列
		return `${this.ename} is working.`;
	}
sal(){
	return `${this.ename}'s salary is ${this.salary}`;
	}
}
```
实现继承：  
```
//实现继承
class Programmer extends Emp{
	constructor(ename,salary,skills){
		super(ename,salary);
		this.skills=skills;
	}
}
var p1=new Programmer("Jack",8800,"PHP&MYSQL");
console.log(p1.ename);
console.log(p1.work());
```
### 8.全局对象  

Node.js中的全局对象是Global  
操作：  

	在Chrome--console中操作；
	写入html操作；
	在node环境中
	cmd的交互环境--可以；
	在脚本中执行--不可以；  

在交互模式下，声明的全局变量是global的成员--会出现全局对象的污染，及多加一些全局变量  
在脚本模式下，声明的全局变量不是global的成员--避免了全局变量的污染  

Global对象的成员属性和成员方法：  

 - console用于向stdout（标准输出）和stderr（标准错误）输出信息。  
 - console.log()   //向stdout输出日志信息  
 - console.info()  //同上  
 - console.error() //向stderr输出错误信息  
 - console.warn()  //同上  
 - console.trace() //向stderr输出栈轨迹信息  
 - console.dir()   //向stdout输出指定对象的字符串表示  
 - console.assert() //断言，为真的断言，错误信息不回输出；为假的断言，会抛出错误对象，输出错误信息，并且终止脚本的运行，可以自定义错误信息  
 - console.time() console.timeEnd()   //测试代码的执行时间，成对出现  

注意：console中的成员方法是异步的，输出顺序和书写书写不一定完全一致。

### 9.process进程  

 - process.arch   //获取cpu架构类型  
 - process.platform   //获取操作系统类型  
 - process.env  //获取操作系统环境变量  
 - process.cwd()  //获取当前所在工作目录  
 - process.execPath  //获取解析器所在目录 
 - process.versions  //获取Node.js的版本 
 - process.uptime()  //获取Node.js解析器运行时间（s） 
 - process.memoryUsage()  //获取内存信息 
 - process.pid   //获取进程id号 
 - process.kill(pid) //向指定进程id号发送退出信号 

### 10.定时器  

 - global.setTimeout()  //一次性定时器  
 - global.setInterval()  //周期性定时器  
 - global.setImmediate()   //在下次事件循环开始之前立即执行的定时器  
 - process.nextTick()   //本次事件循环结束之后立即执行的定时器  

### 11.模块系统：  

Node.js中使用“Module（模块）”来规划不同的功能对象。    
模块的分类：    

 - （1）核心模块--Node.js的内置模块（有些不需引入可以直接使用(console)，有些需要引入（fs））  
- （2）第三方模块  
- （3）自定义模块  
每一个被加载的js文件对应一个模块对象，包含相应的功能和函数。  
模块中声明的变量或函数的作用域叫做“模块作用域”，这些变量和函数都不是global的成员，默认只能在当前的js文件（当前模块）中使用。  
Node.js启动时运行的第一个模块叫做“主模块”--main module，也是自身模块。  
获取主模块对象：  
process.mainModule  
require.main  
除主模块之外的模块都称为子模块。  
默认情况下：某一个模块不能直接使用其他模块中封装的数据，但是每个模块可以导出（exports）自己内部的对象供其他模块使用，也可以引入（require）并使用其他模块导出的对象。  

每一个模块内部都可以使用一个变量：module，指向当前模块自己。  

//判断当前模块是否主模块  
console.log(require.main==module);  

模块的引入：require()    
(在交互模式下，Node.js自带的模块无需引入，直接使用)  
导出模块中的属性和方法供其他模块使用：exports  

预定义的模块作用域成员：  

	__dirname        //当前模块文件所在的目录
	__filename       //当前模块文件所在的目录和文件名
	module           //指向当前模块的应用
	module.exports   //指向当前模块中待导出的供其他模块使用的对象
	exports          //指向module.exports对象的引用
	require          //引入其他模块，使用其他模块的module.exports对象

模块默认代码。底层实现的，是自动加上去的  
```
(function (exports,require,module,__filename,__dirname){
	// module.exports={ };			--模块默认代码 1/3
	// exports=module.exports;		--模块默认代码 2/3

	//求圆的面积和周长
	const PI=3.14;
	var size=function(r){
	return PI*r*r;
	};
	var perimeter=function(r){
	return 2*PI*r;
	};
	
	// console.log(size(6));   //本文件测试
	
	//导出
	exports.size=size;
	exports.p=PI;
	
	// return module.exports;    --模块默认代码  3/3
}
```
自定义模块：  
（1）文件模块  
没有后缀的文件模块，被作为js文件加载  
.js后缀的文件模块，被作为js文件加载  
.json后缀文件模块，被作为JSON字符串加载，会自动解析为对象  
.node后缀文件模块，会被作为C/C++二进制加载  

（2）目录模块--目录模块引入时只需引入目录名即可  
包含一个package.json文件的目录模块  
"main":"xxx"  --指向该模块的主模块  
包含index.js文件的目录模块  
包含index.json文件的目录模块  
包含index.node文件的目录模块  

应该包含一个package.json文件，如果没有就去搜索index.js文件等依次搜索，名字是不能变更的，否则无法使用目录里面的文件了  

放到node_modules目录下的模块，引入的时候直接写模块名称即可，不必指定路径。node_modules目录不必放在当前js文件的目录下，可以放置在node目录的任何一个地方，nodejs会一级一级从当前js文件所在目录往上找  

模块查找的顺序：  
（1）文件/目录模块的缓存  
（2）原生模块的缓存  
（3）原生模块  
（4）文件/目录模块  

### 12.包和npm  

包（package）是根据CommonJS规范，对模块进行的标准封装。  
包规范：  

 - 包是一个目录  
 - 目录中包含一个package.json包说明文件，存放在包顶级目录下，此文件里面的字段可以网上搜索对应的说明  
 - 目录中包含js文件，如有index.js，可以放到包顶级目录下，其它js文件，放到lib目录下  
 - 二进制文件放到bin目录下  
 - 文档放到doc目录下  
 - 单元测试文件放到test目录下  

CommonJS规范要求，包应该位于当前目录或者父目录下的node_modules文件夹下，require函数由近及远依次查找  

NPM包管理器  
[npm官网：www.npmjs.com](www.npmjs.com)  
npm是Node.js的包管理工具，用于下载、安装、升级、删除包或者发布并维护包  
Node.js的安装文件中，已经集成了npm  
下载第三方包：  
（1）安装在当前项目目录下  
`npm install 报名(mysql)`  
会安装到指定目录的node_modules文件夹下（如果nmp比较高版本如6.9.x会把对应的依赖包也下载）
查看目标路径：`npm root`  
（2）安装到全局  
`npm install 包名 -g`  
会安装到npm的目录下  
查看目标路径：`npm root -g`  
列出已经安装的包：`npm ls 包名`  
更新已经安装的包：`npm update 包名`  
卸载已经安装的包：`npm uninstall 包名`  

生成包：  
使用`npm init`目录，可以在当前目录下生成一个package.json文件  

发布包：  
（1）官网注册用户  
（2）使用`npm adduser`命令注册或登录  
(3)进入包所在的目录中配置完成包目录，使用`npm publish`命令发布包  
（4）进入官网搜索以及上传的包  

### 13.Node.js核心模块  

#### 1.console  

- global.consoel 用于向stdout和stderr输出日志信息  
- Console.class console模块中的Console构造函数，用于向任意指定的输出流（文件）中输入信息。  

#### 2.Query String  

提供了处理URI中“查询字符串”部分的相关操作  
--http://www.baidu.com/index.html?a=1&b2#section  

- parse()  //从查询字符串中解析出数据对象，参数为一个查询字符串  
- stringify()   //将一个数据对象反向解析为一个查询字符串格式，参数1，为一个对象；参数2，可选，指定分割符；参数3，可选，指定键值之间分割符  

#### 3.URL模块  

提供了处理url中不同部分的相关操作               
  
- parse()      //解析出url的各个组成部分，参数2，可选，若为true，可以见查询字符串部分解析为对象  
- format()     //将对象反向格式为url格式  
- resolve()    //根据基地址和相对地址解析出目标地址，参数1，基地址；参数2，相对地址  

#### 4.Path模块 
 
提供了对文件路径操作的方法  

- parse()     //解析一个路径，参数为路径字符串  
- format()    //将路径对象格式化为路径字符串，参数为对象  
- join()      //用于连接路径，会正确的使用当前系统的路径分隔符  
- resolv()      //根据基础路径，解析出目标路径的绝对路径  
- relative()    //根据基础路径，获取目标路径与其的相对关系  

#### 5.DNS模块  

提供了域名和IP地址的双向解析功能。  

- lookup()   //把一个域名解析成一个ip地址，从操作系统中查询（缓存）  
- resolve()  //把一个域名解析成一个DNS的记录解析数组，从DNS服务器中查询  
- reverse()   //把一个ip地址反向解析为一组域名  

#### 6.Util工具模块  

- format()  //使用带占位符方式格式化字符串    
- inspect()  //返回一个对象的字符串表示  
- inherits()  //实现构造方法之间的继承  

#### 7.Buffer  

缓冲区，一块专用于存储数据的内存区域，与string类型相对应，用于存储二进制数据。  

- Buffer对象的实例，可以通过读取文件获取，也可以手动创建。  
- Buffer是Global的成员，无需require引入  

#### 8.fs模块--文件系统模块  

fs模块提供了文件的读写、改名、删除、遍历目录等操作。  
fs模块中大多数方法都带有同步和异步两种。  
所有的异步方法都有一个回调函数，此回调函数的第一个参数都是一个错误对象。  
异步方法中如果错误的话，会静默失败，不会自己抛出error，通常情况下都需要自己手动处理。  
常用Class：  

- fs.Stats  //文件或目录的统计信息描述对象  
- fs.ReadStream  //stream.Readable接口的实现对象  
- fs.WriteStream //stream.Writeerable接口的实现对象  
- fs.FSWatcher   //可用于监视文件修改的文件监视器对象  

常用的方法：  

- fs.stat()  //用于返回一个文件或目录的统计信息对象(fs.Stats对象)  
- fs.mkdir()  //创建目录  
- fs.rmdir()   //删除目录  
- fs.dir()   //读取目录下的内容  
- fs.readFile()  //读取文件内容  
- fs.writeFile()  //写文件  
- fs.appendFile()  //追加  
- fs.rename  
- fs.unlink()    //删除  

`---------------------------`  

- fs.stat()   &  fs.statSync()    //用于返回一个文件或目录的统计信息对象(fs.Stats对象)  
- fs.Stats对象的方法：  
- stats.isFile();  
- stats.isDirectory();  

`---------------------------`  

操作目录  

- fs.mkdir()  &  fs.mkdirSync()  
- fs.redir()  &  fs.redirSync()  
- fs.readdir() &  fs.readdirSync()   //读取目录下内容  

`---------------------------`  

操作文件：  

- fs.readFile()  & fs.readFileSync()//读取文件内容  
- fs.writeFile()  &  fs.writeFileSync() //写文件  
- fs.appendFile()  &  fs.appendFileSync() //追加  
- fs.unlink()  & fs.unlinkSync()    //删除  
- fs.rename()    &    fs.renameSync()  
  
#### 9.HTTP模块    

用于构建使用HTTP协议的客户端应用或服务器应用。  
创建并发起请求消息，等待并解析响应消息--实现web客户端  
接收并解析请求消息，构建并发送响应消息--实现web服务器  
常用对象：  

- http.ClientRequest  
- http.Server  
- http.ServerResponse  
- http.IncomingMessage  
- 常用方法：  
- http.createServer  
- http.get()  
- http.request()  

**1.http.request**  
是一个HTTP客户端工具  
用于向web服务器发起一个http请求，并获取响应数据  
有两个主要方法：  

- http.get()  
- http.request()  

该方法返回一个http.ClientRequest对象，用来描述请求信息，回调函数的参数是一个  http.IncomingMessage对象，封装了响应的数据。  
http.ClientRequest对象常用方法：  

- write():向服务器追加请求主体数据  
- end():提交请求消息主体结束  
- setTimeout():设置请求消息超时时间  
- abort():终止请求  
- http.ClientRequest对象常用事件：  
- response:接收到响应消息  
- abort:请求终止事件  
- error:请求发生错误  

注意：使用get()方法不需要手动调用end()方法，使用request()的时候，必须手动调用end()方法。  

**2.http.server**  
http.server是一个基于事件的http服务器  
用于创建web服务器，接收客户端的请求，返回响应消息。所有的请求都被封装到独立的事件当中，我们只需对它的事件编写相应的处理程序，即可实现http的所有功能。  
方法：http.createServer()  
用于创建web服务器应用，可以接收客户端请求，并返回响应消息。  
该方法的返回值是一个对象http.Server。  
http.Server对象的常用方法：  

- listen():启动服务器，监听指定的服务端口  
- close():停止服务器的运行  
- setTimeout():设置服务器响应消息超时时间  
- http.Server对象的常用事件：  
- connection:出现客户端连接  
- request:接收到请求消息    
- close:服务器停止事件  
- error:响应发生错误  
- http.Server对象的request事件回调函数中有两个参数  
第一个参数，http.IncomingMessage对象，封装着客户端提交的请求消息数据  
第二个参数，http.ServerResponse对象，用于构建向客户端输出的响应消息数据。  
  
#### 10.mysql模块 
 
方法：  

- createConnection()  //创建一个mysql服务器的连接，该方法返回一个连接对象，该对象有一下常用方法：
- connect()  //连接数据库
- query()    //提交sql语句给mysql服务器执行
- end()    //断开mysql服务器的连接

命令：  
cmd环境：  

- mysql -h127.0.0.1 -uroot -p -P3306
- show databases;
- source xxxxx.sql;   --执行sql语句
- user xxxx;
- show tables;
- select * from xxx;

#### express模块：  
可以方便的实现服务器的路由、发送和接收http请求，返回响应、发送和接收cookie以及实现模板引擎功能。  

app.method(path,[middleware],function(req,res){})  v