---
title: iOS 网络请求时的一些小套路 
date: 2018-05-25 16:00:00
tags: [iOS]
categories: 卓平
description: 这里有一些自己领悟的套路，还请大家多多提点。
---
#iOS网络请求时的一些个人小套路
这里总结一些过去工作中关于网络请求和返回数据处理时的一些小技巧，提高大家的工作效率。

##网络请求时的参数处理
###糟心例子

```
NSDictionary *params = @{
                             @"arg1":@"value1",
                             @"arg2":@"value2",
                             @"arg3":@"value3",
                             @"arg4":@"value4",
                             @"arg5":@"value5"
                        }; 
                        
//封装的网络请求方法 
[RequestApi  params:params success:^(id res) {
    //返回成功时的处理代码
} failure:^(id res) {
    //返回失败时的处理代码
} error:^(NSError *error) {
    //返回错误时的处理代码
}];                           
```

过去年轻不懂事，请求时的参数是这样设置的。这样的参数字典遍布了整个项目中所有请求的地方，很明显这么写如果接口的参数发生了变化你要改变所有涉及到该请求参数的地方，而且如果key值不小心拼写错误，同样会引发一系列的问题。

###初步的改良
```
+ (NSDictionary *)paramsDicWithArg1:(NSString *)value1 arg2:(NSString *)value2 arg3:(NSString *)value3 arg4:(NSString *)value4 arg5:(NSString *)value5 {
    NSDictionary *params = @{
                             @"arg1":value1,
                             @"arg2":value2,
                             @"arg3":value3,
                             @"arg4":value4,
                             @"arg5":value5
                             };
    //发起网络请求
}
```

以前看到有些开发者用的是这种方式，这种方式是一种简单粗暴的方式。但是...在我以往的开发中动不动就是十几个参数，这么写一个方法的代码特别长，看着心情一下子就不美丽了，所以我一开始就放弃了这种做法。

###当前使用的方法
```
ParamsDicModel.h

@interface ParamsDicModel : NSObject
//在子类的实现中调用该方法确保子类的每一个属性都被赋值
- (BOOL)isValidateParams;
@end


@interface RequestParams : ParamsDicModel
@property (nonatomic, copy)NSString *arg1;
@property (nonatomic, copy)NSString *arg2;
@property (nonatomic, copy)NSString *arg3;
@property (nonatomic, copy)NSString *arg4;
@property (nonatomic, copy)NSString *arg5;

//该方法用于返回处理过后的请求参数
- (NSDictionary *)paramsDic;
@end



//可以把所有请求类的定义放在这个文件中,这样便于查找和管理
@interface OtherRequestParams : ParamsDicModel
...
@end
```

```
ParamsDicModel.m


#import "ParamsDicModel.h"
#import <objc/runtime.h>
@implementation ParamsDicModel

/**
 *	利用OC的runtime机制来判断是否所有属性都被赋值
 *  为什么这么做呢？
 *  得到最终请求参数字典的步骤如下：
 *  1.根据需求定义请求参数类
 *  2.通过请求参数类的实例调用paramsDic返回最终可用的请求参数
 *  isValidateParams方法在paramsDic方法中被调用，用于检查是否有未赋值的参数，从而强制要对所有的属性赋值，这样可以防止参数遗漏，同时也可以避免字典中出现空值的崩溃
 */
- (BOOL)isValidateParams {
    unsigned int outCount, i;
    objc_property_t *properties = class_copyPropertyList([self class], &outCount);
    for (i = 0; i<outCount; i++) {
        objc_property_t property = properties[i];
        const char* char_f = property_getName(property);
        NSString *propertyName = [NSString stringWithUTF8String:char_f];
        id propertyValue = [self valueForKey:(NSString *)propertyName];
        if (propertyValue == nil) {
            NSLog(@"%@属性值为空",propertyName);
            free(properties);
            return NO;
        }
    }
    free(properties);
    return YES;
}
@end


@implementation RequestParams

- (NSDictionary *)paramsDic {
    NSDictionary *params = nil;  
    //判断是否所有属性都已经赋值
    if ([self isValidateParams] == YES) {
        params = @{     
                        @"arg1":self.arg1,
        				@"arg2":self.arg2,
        				@"arg3":self.arg3,
        				@"arg4":self.arg4,
        				@"arg5":self.arg5,
                  };   
    }
    return params;
}

@end

//其他类的实现也是如此
...
```

```
//使用 
                       
RequestParams *requestParams = [RequestParams alloc]init];
requestParams.arg1 = value1;
requestParams.arg2 = value2;
requestParams.arg3 = value3;
requestParams.arg4 = value4;
requestParams.arg5 = value5;
NSDictionary *params = [requestParams paramsDic];  

//封装的网络请求方法 
[RequestApi  params:params success:^(id res) {
    //返回成功时的处理代码
} failure:^(id res) {
    //返回失败时的处理代码
} error:^(NSError *error) {
    //返回错误时的处理代码
}]; 
```

使用该方式的好处是代码的结构更加的清晰，同时也能检测是否有空参数存在，当接口的参数有增删时改动方便，也能通过空参判断来定位某个功能是否有遗漏新增或是减少的某个参数没有处理导致的问题。还有一个优点就是对于那种翻页的请求，其他的参数都不变只有页数发生变化，用这种方法只需改变页数属性值，再调用paramsDic方法就可以，相比之前糟糕的两种方法提升了不少。



##请求返回时的数据处理
处理请求返回数据最常见的做法就是将返回的json数组转换为对应的数据模型，但是...在我以往的开发中 返回数据的层层嵌套，你跟我说你想要创建和处理数据模型，那你是没有见识过什么叫做18层嵌套的绝望。

虽然还没有牛到自己能够很好处理这个问题，但是至少先学会怎么做一个GitHub的搬运工，再学习别人的思路为自己所有。  
  
###两个神器

####第一个神器:[WHCDataModelFactory][1]  
![](/Users/zp/Desktop/融易通/blog/xybank_source/img/zhuoping/1_img1.png)

如果只有第二个神器，数据模型类的定义还是要你一个类一个类、一个属性一个属性的去手动输入，而且多层的嵌套绝对会让你晕头转向，这种吃力不讨好的苦差事还是交给WHCDataModelFactory吧。只需要把服务器返回的json或xml格式数据丢进去，就会自动生成模型类文件。


####第二个神器:[YYModel][2]，是YYKit套件中的一个

在第一步我们用工具生成文件还需根据自己的需求稍加改动才能为自己所有，[YYModel][2]里已经有了详细的介绍，这里就不再赘述。  

```
//使用
[RequestApi  params:params success:^(id res) {
    //返回成功时的处理代码
    CustomClass *customClass = [CustomClass yy_modelWithJSON: res];
    //...业务逻辑
} failure:^(id res) {
    //返回失败时的处理代码
} error:^(NSError *error) {
    //返回错误时的处理代码
}]; 
```

当上述的两个神器用到位了，你就可以脱离繁琐的步骤飞快的完成从服务器返回数据转换成数据模型类的工作，可以说大大提高了数据模型这一层的开发效率。


##总结  
请求参数的处理以及返回数据的处理，这两步现在看来虽然很简单，但对于之前来说也是从开发的实践过程中一个坑一个坑踩出来的。如果本文的能给各位一些小小的启发，那便是我莫大的荣幸了。  
还请路过的大神，能够多多指教，如果文章中什么错误的地方或者有更好的建议请及时指出。



##文末
原理性的东西太大太泛，以我现在功力很难阐述清楚。  
在未来的文章中，我希望提供给大家的是简单有效的事半功倍的开发方式和思路。


###
相信大家对浏览器插件或多或少都有所了解，今天给各位介绍一框浏览器插件[TemperMonkey(油猴)][3]
![](/img/zhuoping/1_img2.jpg)
网站会根据浏览器自动索引适配的浏览器版本，但是谷歌浏览器会跳转到谷歌商店，谷歌商店需要VPN才能登录下载，没有VPN请使用其他浏览器。  
下载完成后按步骤安装即可然后浏览器设置信任就能开启。  
正常开启后就能看到图片中标出的图标，现在光有插件没有脚本。 
![](/img/zhuoping/1_img3.jpg) 
[一个很棒的脚本网站][4]里面有很多大牛贡献的脚本，下载自己需求的脚本，安装到插件中即可使用。脚本可以帮助我们屏蔽掉一些烦人的弹窗广告和百度推荐的垃圾信息，也能帮我们解除一些网站的功能限制，甚至还能实现购物比价的功能。  
无疑，用好插件和脚本，也对提高我们的工作效率有所帮助。如果有会写插件脚本的大神请不吝赐教。







[1]: https://github.com/netyouli/WHC_DataModelFactory
[2]: https://github.com/ibireme/YYModel
[3]: http://tampermonkey.net
[4]: https://greasyfork.org/zh-CN/scripts