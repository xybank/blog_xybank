---
title: java发送手机短信认证码
date: 2018-07-05 20:05:00 
tags: [java] 
categories: 甘明阳
description: java调用ECS短信服务api发送手机认证码
---

一直都有个技术梦想，做出属于自己的门户网站，业余时间都投入网站建设里面，由于网站需要手机认证来实现手机号找回密码功能，对于短信认证码的需求非常强烈，经过几天的研究，终于实现了这个功能(开心)，下面记录一下：

#### 准备工作

​    java上要实现短信认证码的功能，首先要具备一下几点条件：

​    1、实名认证的阿里云账户

​    2、已认证的短信签名和短信模版

​    3、云服务器密钥 

​    4、配置完整的tomcat以及servlet环境

​    5、短信消息API

实名认证的阿里云账户需要你自己去搞定，作为一名有激情的后台开发工程师，云服务器对你的好处非常大，除了短信服务，还有其他好用的功能，像人脸识别，图形功能等等。这里我假定你已经有了可用的阿里云账户。主要介绍剩下的4点：

#### 短信签名和短信模版

首先我们要了解短信签名和短信模版的概念，短信签名是短信前缀，一般是网站名称或者用户名称，短信模版就是短信的内容，如下图，短信签名是"明阳图书"，短信模版就是后面一长串的内容

![Q图片2018070213473](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/java_yzm_01.jpg)

由于互联网环境监管严格，我们网站发送的短信内容都需要结果审核，不然就无法发送哦，下面是具体步骤：

登录阿里云帐号，在搜索栏输入"短信服务"

![Q图片2018070213350](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/java_yzm_02.png)

选择立即开通，由于我已经开通了，样式和你不一样

![Q图片2018070213402](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/java_yzm_03.png)

开通短信服务后找到管理控制台

![Q图片2018070213424](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/java_yzm_04.png)

点击签名管理

![Q图片2018070213531](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/java_yzm_05.png)

点击右上角添加签名

![Q图片2018070213550](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/java_yzm_06.png)

后面的内容也没啥可说的，按照官网提示填写内容，提交审核就行了，一般两个小时就有结果啦，短信模版步骤也是一样的

#### 云服务器密钥

云服务器密钥是包括两个部分，Access Key ID和Access Key Secret，这相当于你阿里云的账户和密码，和普通帐号的使用场景不同，对你的阿里云拥有最高的权限，所以千万不要把自己的密钥透露给别人哦！

点击接口调用，获取AK

![Q图片2018070214011](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/java_yzm_07.png)



点击右上角的创建AccessKey

![Q图片2018070214062](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/java_yzm_07.png)



将创建的密钥保存到桌面，待会备用

#### tomcat以及servlet环境

这个我之前的博客有详细的介绍，[传送门](http://ganmyds.cn/aly_java_tomcat.html)

#### 短信消息API

下载对应语音的消息DEMO工程,工程所需要的所有依赖jar包都放在DEMO工程的lib目录下，将对于的jar包引入到你的工程当中既可按照DEMO样例编写接收消息的程序。阿里云短信服务最少需要两个jar包，aliyun-java-sdk-core和aliyun-java-sdk-dysmsapi，要实现短信工作必须要下这两个jar包。

好啦，所有的准备工作都已经完成，下面我们在java里面调用，按照注释修改即可，无需修改的地方不要动

```java
import com.aliyuncs.DefaultAcsClient;
import com.aliyuncs.IAcsClient;
import com.aliyuncs.dysmsapi.model.v20170525.SendSmsRequest;
import com.aliyuncs.dysmsapi.model.v20170525.SendSmsResponse;
import com.aliyuncs.exceptions.ClientException;
import com.aliyuncs.exceptions.ServerException;
import com.aliyuncs.http.MethodType;
import com.aliyuncs.profile.DefaultProfile;
import com.aliyuncs.profile.IClientProfile;


//发送短信认证码
public class Util_fsdx {
    public static String get_code(String phone) throws ServerException, ClientException{
        //验证码
        String code = "123456";
    	//设置超时时间-可自行调整
    	System.setProperty("sun.net.client.defaultConnectTimeout", "10000");
    	System.setProperty("sun.net.client.defaultReadTimeout", "10000");
    	//初始化ascClient需要的几个参数
    	final String product = "Dysmsapi";//短信API产品名称（短信产品名固定，无需修改）
    	final String domain = "dysmsapi.aliyuncs.com";//短信API产品域名（接口地址固定，无需修改）
    	//替换成你的AK
    	final String accessKeyId = "XXXX";//你的accessKeyId
    	final String accessKeySecret = "XXXX";//你的accessKeySecret
    	//初始化ascClient,暂时不支持多region（请勿修改）
    	IClientProfile profile = DefaultProfile.getProfile("cn-hangzhou", accessKeyId,
    	accessKeySecret);
    	DefaultProfile.addEndpoint("cn-hangzhou", "cn-hangzhou", product, domain);
    	IAcsClient acsClient = new DefaultAcsClient(profile);
    	 //组装请求对象
    	 SendSmsRequest request = new SendSmsRequest();
    	 //使用post提交
    	 request.setMethod(MethodType.POST);
    	 //必填:待发送手机号。支持以逗号分隔的形式进行批量调用，批量上限为1000个手机号码,批量调用相对于单条调用及时性稍有延迟,验证码类型的短信推荐使用单条调用的方式；发送国际/港澳台消息时，接收号码格式为00+国际区号+号码，如“0085200000000”
    	 request.setPhoneNumbers("XXXX");
    	 //必填:短信签名-可在短信控制台中找到
    	 request.setSignName("XXXX");
    	 //必填:短信模板-可在短信控制台中找到
    	 request.setTemplateCode("XXXXX");
    	 //可选:模板中的变量替换JSON串,如模板内容为"亲爱的${name},您的验证码为${code}"时,此处的值为
    	 //友情提示:如果JSON中需要带换行符,请参照标准的JSON协议对换行符的要求,比如短信内容中包含\r\n的情况在JSON中需要表示成\\r\\n,否则会导致JSON在服务端解析失败
    	 request.setTemplateParam("{\"code\":\""+code+"\"}");
    	 //可选-上行短信扩展码(扩展码字段控制在7位或以下，无特殊需求用户请忽略此字段)
    	 //request.setSmsUpExtendCode("90997");
    	 //可选:outId为提供给业务方扩展字段,最终在短信回执消息中将此值带回给调用者
    	 request.setOutId("yourOutId");
    	//请求失败这里会抛ClientException异常
    	SendSmsResponse sendSmsResponse = acsClient.getAcsResponse(request);
    	if(sendSmsResponse.getCode() != null && sendSmsResponse.getCode().equals("OK")) {
    	//请求成功
    		System.out.println("请求成功，验证码是" + code);
    	}
    	return code;
	}
```

#### 说明

由于阿里云只是负责发送给用户手机，不会返回验证码给后台，所以我们需要先用java后台生成短信验证码，可以使用Math类里面的random()生成6个0到10之间的随机整数拼接为一个验证码，这个我也为你写好啦

```java
//生成验证码
    	String code = "";
    	for(int i=0;i<6;i++){
    		code = code + (int)(Math.random()*10);
    	}
```

由于后台很多servlet需要用到短信服务，我建议你将以上代码封装为一个静态方法，以后就可以多次调用了哦！

