---
title: iOS App启动广告思路
date: 2018-05-03 16:00:00
tags: [iOS] 
categories: 洪宇杰 
---

> 很多app(如淘宝、美团等)在启动图加载完毕后，还会显示几秒的广告，一般都有个跳过按钮可以跳过这个广告，有的app在点击广告页之后还会进入一个广告页面，点击返回进入首页。

## 思路
![AppStar](http://p6dtp90fb.bkt.clouddn.com/AppStar.png)


### 1.广告页加载思路  
广告页的内容要实时显示，在无网络状态或者网速缓慢的情况下不能延迟加载，或者等到首页出现了再加载广告页。所以这里我不采用网络请求广告接口获取图片地址，然后加载图片的方式，而是先将图片异步下载到本地，并保存图片名，每次打开app时先根据本地存储的图片名查找沙盒中是否存在该图片，如果存在，则显示广告页。  

```objc
NSData *modelData = [[NSUserDefaults standardUserDefaults] objectForKey:kLaunchADKey];
    LaunchAdModel *model = [LaunchAdModel modelWithJSON:modelData];
    if (model.isShow) {//是否展示广告
        if ([XHLaunchAd checkImageInCacheWithURL:[NSURL URLWithString:model.content]]) {//检查本地是否存在对应图片广告
            [LaunchAdManager showImageAdvertisingWithModel:model];//展示图片广告
        }else if ([XHLaunchAd checkVideoInCacheWithURL:[NSURL URLWithString:model.content]]){//检查本地是否存在对应视频广告
            [LaunchAdManager showVideoAdvertisingWithModel:model];//展示视频广告
        }
    }

```

### 2.判断广告页面是否更新
无论本地是否存在广告图片，每次启动都需要重新调用广告接口。如上面流程图所示，请求API和检测本地广告数据是同时异步进行，互不干扰。根据图片名称或者图片id等方法判断广告是否更新，如果获取的图片名称或者图片id跟本地存储的不一致，则需要重新下载新图片，并删除旧图片。  

```objc
LaunchAdModel *model = [LaunchAdModel modelWithDictionary:json[@"data"]];//模拟广告接口返回数据
        NSURL *url = [NSURL URLWithString:model.content];
        if ([model.type isEqualToString:@"image"]) {
            if ([XHLaunchAd checkImageInCacheWithURL:url]) {//如果本地已经存在该广告就不做更新
                return;
            }
            //更新广告
            [XHLaunchAd downLoadImageAndCacheWithURLArray:@[url] completed:^(NSArray * _Nonnull completedArray) {
                [[NSUserDefaults standardUserDefaults] setObject:[model modelToJSONData] forKey:kLaunchADKey];
                [[NSUserDefaults standardUserDefaults] synchronize];
                [XHLaunchAd clearDiskCacheExceptImageUrlArray:@[[NSURL URLWithString:model.content]]];
                NSLog(@"批量下载缓存图片结果 = %@" ,completedArray);
            }];
        }
```

### 3.广告页点击
如果点击广告需要跳转广告详情页面，那么广告链接地址也需要用NSUserDefaults存储。注意：广告详情页面是从首页push进去的。我们可以将接口返回的数据直接保存在NSUserDefaults中。

### 4.广告显示机制
研究了一下淘宝的广告显示机制，删除淘宝之后重新打开不会显示广告图片，第二次打开才会显示。美团的广告图有时候显示有时候不显示，所以后台在开发广告api的时候可以增加一个字段来判断是否启用广告，如果后台关闭了广告，不显示广告即可。






