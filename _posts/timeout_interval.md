---
title: js中两种定时器的设置及清除
date: 2018-06-28 14:30:55
tags: js中两种定时器的设置及清除
categories: 张杨
typora-root-url: ..
---

1. ##### 周期性定时器

   ```
   function f1(){
   		var n = 5;
   		//启动周期性定时器,每隔1000毫秒让浏览器调用以此函数.
   		var id = setInterval(function(){  //返回该定时器的id
   			console.log(n--);
   			//当n<0时停止计时器
   			if(n<0){
   				console.log("爆炸!");
   				clearInterval(id);
   			}
   		},1000) 
   		/*
   			启动的定时器类似于一个子线程,当前的方法f1()类似于主线程<main),
   			而这并发执行,即上线程启动完子线程后,立刻输出boom,而子线程却在1秒后执行.
   		*/
   		console.log("boom!");
   	}
   ```

2. ##### 一次性定时器

    ```
    //一次性定时器
    	var id;
    	function f2(){
    		//启动一次性定时器,在5000毫秒后让浏览器调用函数.
    		//调用完成后,定时器会自动结束.也可以在未调用前,手动结束.
    		id = setTimeout(function(){   //返回该定时器的id
    			console.log("啪!");
    		},5000);
    	}
    	
    	//取消
    	function f3(){
    		clearTimeout(id);
    	}
    ```

    

   

