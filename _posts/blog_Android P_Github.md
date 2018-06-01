---
title: Android_P适配总结
date: 2018-06-01 13:00:00
tags: [Android P,兼容适配]
categories: 黄涛
description: 本文是Android客户端小组对华为提供的Android P适配文档进行调研、学习及总结
---

## Android P适配

### Android P版本取消预置Apache Http客户端jar包

该适配需要客户端本地预置apache http客户端包，目前项目中已引用
    `compile 'org.jbundle.util.osgi.wrapped:org.jbundle.util.osgi.wrapped.org.apache.http.client:4.1.2'`
该引用为远程依赖apache http jar包，从测试结果来看满足适配要求。


### Android P版本targetSdkVersion小于17适配

该适配已完成，目前项目targetSdkVersion为21
```

    compileSdkVersion 25
    buildToolsVersion '25.0.2'
    // 默认设置
    defaultConfig {
    applicationId "com.cib.cibmb"
    minSdkVersion 14
    targetSdkVersion 21
    versionCode 38
    versionName "4.0.2
```

### Android P版本非SDK管控适配

非官方SDK接口分为3类：

1. 浅灰名单
2. 深灰名单
3. 黑名单

其中浅灰名单和深灰名单不适配也不会存在任何问题，而且在Android P设备上测试登录、转账、理财等页面没有弹出该类警告弹框

黑名单接口见下：
```

    Ldalvik/system/VMRuntime;->setHiddenApiExemptions([Ljava/lang/String;)V
    Ljava/lang/invoke/MethodHandles$Lookup;->IMPL_LOOKUP:Ljava/lang/invoke/MethodHandles$Lookup;
    Ljava/lang/invoke/VarHandle;->acquireFence()V
    Ljava/lang/invoke/VarHandle;->compareAndExchange([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->compareAndExchangeAcquire([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->compareAndExchangeRelease([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->compareAndSet([[Ljava/lang/Object;)Z
    Ljava/lang/invoke/VarHandle;->fullFence()V
    Ljava/lang/invoke/VarHandle;->get([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->getAcquire([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->getAndAdd([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->getAndAddAcquire([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->getAndAddRelease([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->getAndBitwiseAnd([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->getAndBitwiseAndAcquire([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->getAndBitwiseAndRelease([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->getAndBitwiseOr([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->getAndBitwiseOrAcquire([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->getAndBitwiseOrRelease([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->getAndBitwiseXor([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->getAndBitwiseXorAcquire([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->getAndBitwiseXorRelease([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->getAndSet([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->getAndSetAcquire([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->getAndSetRelease([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->getOpaque([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->getVolatile([[Ljava/lang/Object;)Ljava/lang/Object;
    Ljava/lang/invoke/VarHandle;->loadLoadFence()V
    Ljava/lang/invoke/VarHandle;->releaseFence()V
    Ljava/lang/invoke/VarHandle;->set([[Ljava/lang/Object;)V
    Ljava/lang/invoke/VarHandle;->setOpaque([[Ljava/lang/Object;)V
    Ljava/lang/invoke/VarHandle;->setRelease([[Ljava/lang/Object;)V
    Ljava/lang/invoke/VarHandle;->setVolatile([[Ljava/lang/Object;)V
    Ljava/lang/invoke/VarHandle;->storeStoreFence()V
    Ljava/lang/invoke/VarHandle;->weakCompareAndSet([[Ljava/lang/Object;)Z
    Ljava/lang/invoke/VarHandle;->weakCompareAndSetAcquire([[Ljava/lang/Object;)Z
    Ljava/lang/invoke/VarHandle;->weakCompareAndSetPlain([[Ljava/lang/Object;)Z
    Ljava/lang/invoke/VarHandle;->weakCompareAndSetRelease([[Ljava/lang/Object;)Z
```

搜索项目代码和产品源码暂时没有发现有这些方法进行反射调用的地方，而且经过Android P设备测试也能正常运行，故暂时没有发现这块需要适配的地方。



### Android P 项目中测试发现兼容性问题

除了上述华为官方提供适配文档外，在使用Android P设备测试时发现Build.VERSION.RELEASE返回手机系统版本号为P，之前返回的都是类似8.0.0、7.1.1的字符串，导致代码中进行截取会出现异常导致闪退等问题，产品组意见为Android P版本正式发布前再跟进此问题，项目中为了适配，在4.0.2客户端代码进行优化

```
    "P".equalsIgnoreCase(Build.VERSION.RELEASE) ? "9.0.0" : Build.VERSION.RELEASE
```

判断当前版本号是否匹配P or p，如果匹配则替换为9.0.0字符串，防止进行截取出现异常。



