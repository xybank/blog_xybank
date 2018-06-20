---
title:  关于ios指纹登陆功能ewp部分简单描述

date: 2018-06-13 14:50:52

tags:

---

### 设置指纹：

要使用指纹登陆必须得先开通指纹登陆功能，所以我们需要先设置指纹开通，调用客户端给的bol,ecode=native：canAuthenticate（）方法检测系统是否有设置指纹密码，该方法会返回两个值，当bol=false，ecode=-7时会提示用户尚未设置Touch iD，需要先在手机上设置touch id。bol=true时表示可以开通指纹功能。
<!--more-->

 ```js
bol,eCode =native:canAuthenticate();
if bol == false and eCode == -7  then
    alert("您尚未设置Touch ID，请先在手机系统“设置-Touch ID与密码”中添加指纹。");
elseif bol == true then  --access_token开通指纹时生成
    --可以做开通指纹的方法
end
 ```
开通指纹时产生的access_token会保存在客户端本地，调用客户端方法native:storeKeyAndSecret保存。
```
--请求接口开通指纹，接口会返回access_token，yxts（有效天数），rzlx（认证类型）
access_token=access_token["value"];
yxts=yxts["value"];
if database:getData(userid) == "" or database:getData(userid) == nil then
	zhiwen_password=yxts.."|"..FRM_time.."|"..rzlx;
else                 					  	          			   zhiwen_password=database:getData(userid).."rytong"..yxts.."|"..FRM_time.."|"..rzlx;
end
native:storeKeyAndSecret("access_token_key", access_token, 	"access_token_serviceData");--ios把access_token用 storeKeyAndSecret保存
database:addData(userid, zhiwen_password);
```
还要把指纹有效期（90天）、设置开通指纹时的时间、登陆认证类型（主要是区分指纹还是手势）拼接成一个字段进行保存（zhiwen_password），

## 登陆验证指纹：

设置完用指纹登陆后，这个时候回到登陆界面就可以用指纹进行登录了

登陆验证时先验证有效期过了没有，如果设置指纹时间超过90天则报错说

`因您长时间未使用指纹登录，请先通过验证密码登录，成功登录后下次登录时可继续使用指纹登录。`

反之进行下一步验证，调用客户端方法native:authenticate()，验证时会有传两个值给后台，方案图如下：

![qqq](/img/lizd/zw.png)

部分代码：

```
if val1 == false then
    if val2 == -3 or val2 == -6 or val2 == -10 or val2 == -1004 then  --点击验证密码, -6是该设备没有指纹功能,-10会话不可用，-1004身份验证失败，因为他需要显示已被禁止的UI
    elseif val2 == -2 or val2 == -9 or val2 == -4 then -- -2点击取消,-9按home键取消，-4来电话的情况
    elseif val2 == -7 or val2 == -5 then -- 没有设置指纹   -5关闭了指纹
    elseif val2 == -8 then -- -8被锁了
    elseif val2 == -1 then  -- -1, 错误输入超过三次
    elseif val2 == 100 then  --您设备的TouchID发生了变化，请使用密码登录。
    end
elseif val1 == true then  --验证指纹成功
end
```
具体方法就不贴出来了，不同的app可能走的方法也不同，只是把大概的流程判断给贴出来。

验证成功的那一步还需要做个判断，因为有可能之前保存在客户端的access_token因为某些原因丢失，所以要判断access_token是否为空，如果为空，那就只能提示用户使用密码登陆了。

```
local access_token=native:getSecretByKey("access_token_key", "access_token_serviceData");  --取出保存在客户端的access_token
if access_token == "" or access_token == nil then
	    local  userid=gzjzl..gzjhm;              --取不到的话就提示用户用密码登录
		cmm_unit_fun.public:alert("指纹密钥已丢失,请使用密码登录","确定",
        function(index)
        changelogin();
        location:reload();
        end);
	else
        native:storeKeyAndSecret("access_token_key", access_token, "access_token_serviceData");--ios把access_token用 storeKeyAndSecret保存
       --这里写把值传给接口的方法 
     end
else
   --这里写把值传给接口的方法
end
```



