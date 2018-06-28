---
title: erlang手记 & 一个版本问题
date: 2018-06-22 23:07:43
tags: [erlang]
categories: 余声赞
description: erlang手记 & 一个版本问题
---

## 1.前言  

重温erlang基础，把一些常用以及牛逼哄哄的知识点记录下来。当然了，重温的内容包括却不止于下面的内容。   

<!--more-->

## 2.知识点列表  

- **查看自己的pid**  
    self()  
- **查看已经启动的进程**  
    i()  
- **-module(person)**  
    person应该和文件夹名一致，是原子  
- **进程间发送消息格式**  
    Pid ! Msg  
- **接收进程代码**  
``` erlang
    receive  
        {From,Message}->  
        ...  
    end
```  
    如在erlang shell中：receive X -> X end. %打印接收的消息
- **spawn**  
``` erlang
    Pid = spawn(io,format,["danz=====~n~p~n",["is very good"]])  
```  
- **并发**  
    并发的好处：  
    1.性能：性能能够得到很好的提升，现在多核cpu已经普及，成本更低了；  
    2.可拓展性：erlang的拓展性天生就很好；  
    3.容错：进程间互不干扰，所以一个进程崩溃了不会影响其他进程  

    并发和并行：  
    并发和软件有关，如erlang是并发编程语言，有描述并发的语言结构，能够编写并发的程序  
    并行和硬件有关，如并行计算机是一种有多个处理单元（cpu或核心）运行的计算机  

    基于操作系统的并发和基于语言的并发：  
    基于操作系统的并发，程序在不同操作系统上会有不同的工作方式；erlang的并发在所有操作系统有相同的工作方式，是基于语言的并发    
- **官网**  
    http://www.erlang.org/  
- **打开erlang shell**  
    cmd或者linux中：  
    erl  
- **清除shell绑定的变量**  
    f()  
- **停止erl shell**  
    q()是init:stop()别名  
    erlang:halt()  
- **erlang shell查看目录**  
``` erlang
    pwd()  
    ls()   
    cd(Dir)  
```   
- **Windows环境下查看erlang的源码**  
    Windows环境下查看erlang的源码：C:\Program Files (x86)\erl5.9.3.1\lib\stdlib-1.18.3\src  
    然还有很多其它的模块  
- **lists模块**  
``` erlang
    lists:map(F,List)  
    lists:filter()  
    lists:seq(1,N)   
    lists:reverse([1,2,3,4])   
    ... 
```   
- **小程序**  

  排序：  
``` erlang
  qsort([])->[];  
  qsort([Pivot|T])->  
    qsort([X || X <- T,X < Pivot])  
    ++ [Pivot] ++  
    qsort([X || X <- T,X >= Pivot]).
```   

  勾股定理数：  
``` erlang
    ggdl(N)->
      [{A,B,C} || 
      A <- lists:seq(1,N),
      B <- lists:seq(1,N),
      C <- lists:seq(1,N),
      A*A+B*B=:=C*C
      ].
```   

  回文构词：  
``` erlang
    perms([])->[[]];
    perms([L])->[[H|T] || H <- L, T <- perms(L -- [H])].
```   

- **打印**  
``` erlang
    io:format("~n~p~n",[Paras]).    
    io:format("~n~w~n",[Paras]).    
    io:format("~n~t~n",[Paras]).      %可以打印中文
```   
- **记录 & 映射组**  
    记录：

  定义：`-record{todo,{status=reminder,who=joe,text}}.`

  记录本质就是元组。

  shell里面读取记录命令：`rr("todo.hrl") `
  shell忘记记录命令：`rf(todo)`      %忘记记录定义后，输出已经定义的记录，发现本质就是元组

  %判断X是否是todo类型记录
``` erlang
  do_something(X) when is_record(X, todo)->
```   

  映射组：

  定义：`#{a => 1,b => 2}   #{a := 1,b := 2}`

  统计字符出现次数：<font color="red">(注意要在R17版本以及以后版本才能使用此功能)</font>
``` erlang
  count_chars(Str)->
    count_chars(Str,#{}).
  count_chars([H|T],#{H => N}=X) ->
    count_chars(T,X#{H := N+1});
  count_chars([H|T],X) ->
    count_chars(T,X#{H => 1});
  count_chars([],X) ->
    X. 
```   
- **异常**  
    ①：try ... catch ... after ... end
  ②：catch 

  主动抛出：
``` erlang
  throw(Exception)    
  exit(Exception)    
  error(Exception)    
```   
  得到栈信息：
``` erlang
  erlang:get_stacktrace()  
```   
- **二进制型(binary)和位串(bitstring)**  
    例子1：
``` erlang
  Red=2
  Green=61
  Blue=20
  M = <<Red:5,Green:6,Blue:5>>.  %%<<23,180>>
```   
  例子2：
``` erlang
  B1 = <<1:8>>
  byte_size(B1)  %% 1
  is_binary(B1)  %% true
  is_bitstring(B1)  %% true
  B2 = <<1:17>>  %%<<0,0,1:1>>
  is_binary(B2) %%false
  is_bitstring(B2)  %% true
  byte_size(B2) %%3
  bit_sise(B2) %% 17  
```   
- **提取模块信息方式**  
    ModuleNmae:module_info()  %如：lists:module_info()  
  
## 3.erlang版本问题  

在《Programming Erlang》第2版第5章有个例子，计算字符出现次数，代码如下：
``` erlang
-module(countChar).  
-export([count_characters/1]).  
count_characters(Str) ->  
    count_characters(Str, #{}).  
count_characters([H|T],  #{ H => N } = X) ->  
    count_characters(T, X#{ H := N+1 });  
count_characters([H|T], X) ->  
    count_characters(T, X#{ H => 1 });  
count_characters([], X) ->   
    X.  
```   
在Windows环境中的cmd下编译出错：
``` bash
C:\Users\danz\erl>erlc lib_misc.erl
c:/Users/danz/erl/lib_misc.erl:31: syntax error before: '{'
c:/Users/danz/erl/lib_misc.erl:34: syntax error before: '{'
c:/Users/danz/erl/lib_misc.erl:2: function count_characters/1 undefined
```   
编译其它erl文件，得到正确的执行：
``` erlang
C:\Users\danz\erl>erlc helloworld.erl

C:\Users\danz\erl>erl -noshell -s helloworld start -s init stop
helloworld2
C:\Users\danz\erl>
```   
搜索相关错误，如搜索：count_characters.erl，得到以下文章：https://blog.csdn.net/hzhsan/article/details/50197815
文章中说的很明白，原因：

在 Erlang/opt 17中
- No variable keys (key中不能含变量) 
- No single value access (不支持取单个value，即不支持#{Key}操作) 
- No map comprehensions (不支持map推导，例如 #{X => foggy || X <- [london,boston]}.)

18 中仅得到部分支持
如果我们要使用上述功能，应该运用 Erlang 提供给我们的 maps 模块。
``` erlang
maps:new() 
erlang:is_map(M) 
maps:is_key(Key, Map) 
maps:get(Key, Map) 
...
```   

知道是因为erlang版本的原因，故在上提到的文章中下载OTP 18版本（或者去官网下载），安装到本机中，那么此时本机上有两个版本的erlang了，分别是：
Erlang/OTP R15B03；
Erlang/OTP 18
（
<font color="BlueViolet">在Windows中查看erlang的安装的版本方法</font>：  
①找到对应的安装路径，如：C:\Program Files (x86)\erl7.1，打开里面doc文档的index.html，文档头部有标明版本信息；
②打开erlang shell，注意不是在cmd环境中打开，是erlang自带的shell；
<font color="BlueViolet">如果是在linux系统中</font>，当输入erl时，就把对于的版本信息都打印出来了。在Windows下cmd输入erl，没有打印全部版本信息
）
本机Windows环境下，我想在cmd中使用OTP 18版本，原来配置了OTP R15B03的环境变量，可以手动配置OTP 18环境变量，或者暂时在一个cmd中测试，步骤：
cmd中，临时加入路径：
``` erlang
C:\Users\danz\erl>set PATH="C:\Program Files (x86)\erl7.1\bin";%PATH%
```   
C:\Program Files (x86)\erl7.1\bin是OTP 18安装路径；
继续操作：
``` erlang
C:\Users\danz\erl>erl
Eshell V7.1  (abort with ^G)
1> c(lib_misc).
{ok,lib_misc}
2> lib_misc:count_characters("helloworld").
#{100 => 1,101 => 1,104 => 1,108 => 3,111 => 2,114 => 1,119 => 1}
```   
得到正确执行！
打开另外一个cmd，查看版本：
``` erlang
C:\Users\danz\erl>erl
Eshell V5.9.3.1  (abort with ^G)
1> c(lib_misc).
lib_misc.erl:31: syntax error before: '{'
lib_misc.erl:34: syntax error before: '{'
lib_misc.erl:2: function count_characters/1 undefined
error
```  
是原来版本，编译错误。 

## 4.erlang自带shell编译问题  

问题表现：  

![](/img/yushengzan/erlang-shouji1.png)  

搜索错误信息：lib_misc.bea#: error writing file，得到文章：https://stackoverflow.com/questions/5495756/how-to-use-erlang-examples
里面提到：<font color="red">To compile a .erl file, the compiler needs to be able to write out the compiled .beam file.</font>
换句话说就是没有创建文件的权限，就是权限问题，故使用管理员身份运行erlang shell：

![](/img/yushengzan/erlang-shouji2.png)  

问题解决。

遗留问题：
   怎么改变erlang自带shell的工作目录？