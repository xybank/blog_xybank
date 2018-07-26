---
title: Mini Program初探
date: 2018-07-26 11:00:00
tags: [微信小程序,学习]
categories: 黄涛
description: 闲暇之余研究下微信小程序开发
---

## 微信小程序

### 认知篇

微信小程序（wei xin xiao cheng xu），简称小程序，英文名Mini Program，是一种不需要下载安装即可使用的应用，它实现了应用“触手可及”的梦想，用户扫一扫或搜一下即可打开应用。
全面开放申请后，主体类型为企业、政府、媒体、其他组织或个人的开发者，均可申请注册小程序。小程序、订阅号、服务号、企业号是并行的体系。
2017年1月9日，张小龙在2017微信公开课Pro上发布的小程序正式上线。
2018年2月，微信官方发布公告称：已对涉及假货高仿、色情低俗和违规“现金贷”等超过2000个小程序，进行永久封禁处理。

以上内容摘自百度百科

### 工具篇
微信开发工具下载地址：[https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html?t=2018724](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html?t=2018724)
安装成功后可以新建一个项目，见下图：

![](http://bmob-cdn-18057.b0.upaiyun.com/2018/07/26/d48faa2c40693f0b80d67cd63101ee7a.png)

其中AppID需要申请小程序开发账号，填写资料提交审核后可以获取


### 开发篇

目录结构浏览

![](http://bmob-cdn-18057.b0.upaiyun.com/2018/07/26/9833a2f54032552e80a03585dba3b855.png)

.json JSON配置文件：逻辑层 主要和UI交互进行事件的处理及触发UI页面的渲染等操作；

.wxml WXML模板文件：UI层 渲染页面，和逻辑层进行交互，触发页面重绘或者渲染等操作；

.wxss WXSS样式文件： UI层样式控制器

其中 app.json用于配置全局环境设置，见代码

   

	    {
	      "pages":[
	    "pages/index/index",
	    "pages/logs/logs"
	      ],
	      "window":{
	    "backgroundTextStyle":"light",
	    "navigationBarBackgroundColor": "#fff",
	    "navigationBarTitleText": "WeChat",
	    "navigationBarTextStyle":"black"
	      }
	    }



pages中放在数组第一位的则是启动页，window中主要是运行环境及状态的配置。


获取用户信息



	  	getUserInfo:function(cb){
	    var that = this
	    console.log(1);
	    if(this.globalData.userInfo){
	      
	      typeof cb == "function" && cb(this.globalData.userInfo)
	    }else{
	      //调用登录接口
	      wx.login({
	        success: function () {
	          wx.getUserInfo({
	            success: function (res) {
	              that.globalData.userInfo = res.userInfo
	              typeof cb == "function" && cb(that.globalData.userInfo)
	            }
	          })
	        }
	      })
	    }
 	 },



简单一句代码即可获取当前用户信息，其中包括微信名称、头像图片地址、城市、性别等。

最后通过以下代码，即可达到刷新用户UI页面渲染的效果

    
	    //调用应用实例的方法获取全局数据
	    app.getUserInfo(function(userInfo){
	      //更新数据
	      that.setData({
	    userInfo:userInfo
	      })
	    })


### 总结篇
微信小程序的优势在于依附于微信，除了庞大的用户红利之外，对于使用者来说也是非常方便的，不需要过多的手机内存或性能消耗来使用应用。开发起来也很方便，不论是调试还是真机运行都很快捷。最后建议有想法、有追求的同事可以亲身体会下小程序开发，会有意向不到的收获的。