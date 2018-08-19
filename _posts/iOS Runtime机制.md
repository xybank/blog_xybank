---
title: iOS Runtime机制
date: 2018-08-18
tags: [iOS]
categories: 卓平
description: 本文主要描述iOS Runtime机制的基础知识
---


Runtime机制是Objective-C面向对象和动态机制的基石。Objective-C 是一个动态语言这意味着它不仅需要一个编译器，也需要一个运行时系统来动态得创建类和对象、进行消息传递和转发。


## Runtime的数据结构

###id (obj_object)
在OC中,使用[receiver message]语法不会马上执行对象的message方法而是向receiver发送了一条message消息，这条消息可能由receiver处理，也可能转发给其他对象，还有可能接收到消息但是不处理。receiver会被编译器转化成


``` OC 
id objc_msgSend(id self, SEL op, ...);

//self:接收消息的类实例的指针[receiver message]中的receiver
//op:处理消息的方法选择器
//...:包含参数的可变参数列表
```

通过下面这个runtime库的源码我们就能知道我们经常用到的id是什么了。

``` OC 
/// Represents an instance of a class.
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;
```

id其实就是一个指向objc_object结构体指针，它包含一个Class isa成员，根据isa指针就可以顺藤摸瓜找到对象所属的类。

> **注意**：根据Apple的官方文档[Key-Value Observing Implementation Details][1]提及，key-value observing是使用isa-swizzling的技术实现的，isa指针在运行时被修改，指向一个中间类而不是真正的类。所以，你不应该使用isa指针来确定类的关系，而是使用class方法来确定实例对象的类。

###isa (objc_class)
isa指针的数据类型是Class，Class表示对象所属的类。

``` OC 
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;

/// An opaque type that represents an Objective-C class.
typedef struct objc_class *Class;
```

> **注意**：OBJC2_UNAVAILABLE是一个Apple对Objc系统运行版本进行约束的宏定义，主要为了兼容非Objective-C 2.0的遗留版本，但我们仍能从中获取一些有用信息。

让我们分析一些重要的成员变量表示什么意思和对应使用哪些数据结构。

- **isa** 

在此处的isa表示Class对象的Class，被称为元类。Class本身也是一个对象，objc_class有以下定义证实了这一点。
	
``` OC
struct objc_class : objc_object {
  	// Class ISA;
  	Class superclass;
  	cache_t cache;             // formerly cache pointer and vtable
  	class_data_bits_t bits;    // class_rw_t * plus 	custom rr/alloc flags
  	......
}
```

- **super_class**: 实例对象对应的父类
- **name**:类名
- **ivars**:表示多个成员变量，它指向objc\_ivar\_list结构体。

``` OC
struct objc_ivar_list {
  int ivar_count                                           OBJC2_UNAVAILABLE;
#ifdef __LP64__
  int space                                                OBJC2_UNAVAILABLE;
#endif
  /* variable length structure */
  struct objc_ivar ivar_list[1]                            OBJC2_UNAVAILABLE;
}

```

objc\_var\_list是一个链表，存储多个objc\_ivar，objc\_ivar结构体存储类的当个成员变量的信息

- method_lists表示方法列表，它指向objc_ method_list结构体的二级指针，可以动态修改*methodLists的值来添加成员方法，也是Category实现原理，同样也解释Category不能添加属性的原因。

``` OC
struct objc_method_list {
  struct objc_method_list *obsolete                        OBJC2_UNAVAILABLE;

  int method_count                                         OBJC2_UNAVAILABLE;
#ifdef __LP64__
  int space                                                OBJC2_UNAVAILABLE;
#endif
  /* variable length structure */
  struct objc_method method_list[1]                        OBJC2_UNAVAILABLE;
}
```
objc\_method\_list也是一个链表，存储多个objc\_method，objc\_method结构体存储类的某个方法的信息。

- **cache**: 用来缓存访问经常访问的方法，它指向objc\_cache结构体。

通常一个类只有20%的方法会经常被调用，因此用cache来提高效率，避免每次都要遍历method\_lists来查找要调用的方法，只有在cache中找不到时，才在method\_lists查找。

``` OC
typedef struct objc_cache *Cache                             OBJC2_UNAVAILABLE;

struct objc_cache {
    unsigned int mask /* total = mask + 1 */                 OBJC2_UNAVAILABLE;
    unsigned int occupied                                    OBJC2_UNAVAILABLE;
    Method buckets[1]                                        OBJC2_UNAVAILABLE;
};
```

- **protocols**:  表示类遵循的哪些协议。


### Method (objc_method)

Method表示类中的某个方法，定义如下

``` OC
/// An opaque type that represents a method in a class definition.
typedef struct objc_method *Method;
struct objc_method {
    SEL method_name                                          OBJC2_UNAVAILABLE;
    char *method_types                                       OBJC2_UNAVAILABLE;
    IMP method_imp                                           OBJC2_UNAVAILABLE;
}
```

其实Method就是一个指向objc\_method结构体指针，它存储了方法名(method\_name)、方法类型(method\_types)和方法实现(method\_imp)等信息。而method_imp的数据类型是IMP，它是一个函数指针，后面会重点提及。

### Ivar (objc_ivar)
Ivar表示类找那个的实例变量，定义如下

``` OC
/// An opaque type that represents an instance variable.
typedef struct objc_ivar *Ivar;

struct objc_ivar {
    char *ivar_name		OBJC2_UNAVAILABLE;
    char *ivar_type		OBJC2_UNAVAILABLE;
    int ivar_offset		OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space			OBJC2_UNAVAILABLE;
#endif
}
```

Ivar是一个指向objc\_ivar结构体指针，它包含了变量名(ivar\_name)、变量类型(ivar\_type)等信息。


### IMP
在Method中，其本质是一个函数指针,指向方法的实现，定义如下

``` OC
/// A pointer to the function of a method implementation. 
#if !OBJC_OLD_DISPATCH_PROTOTYPES
typedef void (*IMP)(void /* id, SEL, ... */ ); 
#else
typedef id (*IMP)(id, SEL, ...); 
#endif
```
当我们向某个对象发送消息时，可以由这个指针来指定方法的实现，它最终就会执行那段代码，这样就可以绕开消息传递阶段而去执行另一个方法实现。


## Runtime的消息传递

### objc_msgSend它具体是如何发送消息：

1. 首先根据receiver对象的isa指针获取它对应的class；
2. 优先在class的cache查找message方法，如果找不到，再到methodLists查找；
3. 如果没有在class找到，再到super_class查找；
4. 一旦找到message这个方法，就执行它实现的IMP。


### self与super

看以下例子会输出什么？

``` OC
@implementation Son : Father
- (id)init
{
    self = [super init];
    if (self)
    {
        NSLog(@"%@", NSStringFromClass([self class]));
        NSLog(@"%@", NSStringFromClass([super class]));
    }
    return self;
}
@end
```

self表示当前这个类的对象，而super是一个编译器标示符，和self指向同一个消息接受者。在本例中，无论是[self class]还是[super class]，接受消息者都是Son对象，但super与self不同的是，self调用class方法时，是在子类Son中查找方法，而super调用class方法时，是在父类Father中查找方法。

当调用[self class]方法时，会转化为objc_msgSend函数。

这时会从当前Son类的方法列表中查找，如果没有，就到Father类查找，还是没有，最后在NSObject类查找到。

所以NSLog(@"%@", NSStringFromClass([self class]));会输出Son。


当调用[super class]方法是，会转化为objc\_msgSendSuper函数。
objc\_msgSendSuper定义如下

``` OC
/// Specifies the superclass of an instance. 
struct objc_super {
    /// Specifies an instance of a class.
    __unsafe_unretained id receiver;

    /// Specifies the particular superclass of the instance to message. 
#if !defined(__cplusplus)  &&  !__OBJC2__
    /* For compatibility with old objc-runtime.h header */
    __unsafe_unretained Class class;
#else
    __unsafe_unretained Class super_class;
#endif
    /* super_class is the first class to search */
};
#endif

```

结构体包含两个成员，第一个是receiver，表示某个类的实例。第二个是super\_class表示当前类的父类。

这时首先会构造出objc_super结构体，这个结构体第一个成员是self，第二个成员是(id)class\_getSuperclass(objc\_getClass("Son"))，实际上该函数会输出Father。然后在Father类查找class方法，查找不到，最后在NSObject查到。此时，内部使用objc\_msgSend(objc\_super->receiver, @selector(class))去调用，与[self class]调用相同，所以结果还是Son。

### 隐藏参数self和_cmd

当[receiver message]调用方法时，系统会在运行时偷偷地动态传入两个隐藏参数self和\_cmd，之所以称它们为隐藏参数，是因为在源代码中没有声明和定义这两个参数。至于对于self的描述，上面已经解释非常清楚了，下面我们重点讲解\_cmd。

\_cmd表示当前调用方法，其实它就是一个方法选择器SEL。一般用于判断方法名或在Associated Objects中唯一标识键名。


## 方法解析与消息转发

[receiver message]调用方法时，如果在message方法在receiver对象的类继承体系中没有找到方法，那怎么办？一般情况下，程序在运行时就会Crash掉，抛出unrecognized selector sent to…类似这样的异常信息。但在抛出异常之前，还有三次机会按以下顺序让你拯救程序。

1. Method Resolution
2. Fast Forwarding
3. Normal Forwarding


### Method Resolution
 
首先Objective-C在运行时调用+ resolveInstanceMethod:或+ resolveClassMethod:方法，让你添加方法的实现。如果你添加方法并返回YES，那系统在运行时就会重新启动一次消息发送的过程。

举一个简单例子，定义一个类Message，它主要定义一个方法sendMessage，下面就是它的设计与实现

``` OC
@interface Message : NSObject
- (void)sendMessage:(NSString *)word;
@end

@implementation Message
- (void)sendMessage:(NSString *)word
{
    NSLog(@"normal way : send message = %@", word);
}
@end
```

以上代码若创建实例调用方法，结果一目了然。

但如果注释掉原来的sendMessage方法，覆盖resolveInstanceMethod方法：

``` 0C
+ (BOOL)resolveInstanceMethod:(SEL)sel
{
    if (sel == @selector(sendMessage:)) {
        class_addMethod([self class], sel, imp_implementationWithBlock(^(id self, NSString *word) {
            NSLog(@"method resolution way : send message = %@", word);
        }), "v@*");
    }
    return YES;
}
```
此时输出的结果与原来调用sendMessage:方法时是一样的

> **注意**: 上面代码有这样一个字符串"v@*，它表示方法的参数和返回值，详情请参考[Type Encodings][2]。

### Fast Forwarding
如果目标对象实现- forwardingTargetForSelector:方法，系统就会在运行时调用这个方法，只要这个方法返回的不是nil或self，也会重启消息发送的过程，把这消息转发给其他对象来处理。否则，就会继续Normal Fowarding。

继续上面Message类的例子，将sendMessage和resolveInstanceMethod方法注释掉，然后添加forwardingTargetForSelector方法的实现：

``` OC

- (id)forwardingTargetForSelector:(SEL)aSelector
{
    if (aSelector == @selector(sendMessage:)) {
        return [MessageForwarding new];
    }
    return nil;
}

```

MessageForwarding类需要另外创建并写好实例方法sendMessage:

这里叫Fast，是因为这一步不会创建NSInvocation对象，但Normal Forwarding会创建它，所以相对于更快点。


### Normal Forwarding

如果没有使用Fast Forwarding来消息转发，最后只有使用Normal Forwarding来进行消息转发。它首先调用methodSignatureForSelector:方法来获取函数的参数和返回值，如果返回为nil，程序会Crash掉，并抛出unrecognized selector sent to instance异常信息。如果返回一个函数签名，系统就会创建一个NSInvocation对象并调用-forwardInvocation:方法。

继续前面的例子，将forwardingTargetForSelector方法注释掉，添加methodSignatureForSelector和forwardInvocation方法的实现：

``` OC

- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector
{
    NSMethodSignature *methodSignature = [super methodSignatureForSelector:aSelector];
    if (!methodSignature) {
        methodSignature = [NSMethodSignature signatureWithObjCTypes:"v@:*"];
    }
    return methodSignature;
}
- (void)forwardInvocation:(NSInvocation *)anInvocation
{
    MessageForwarding *messageForwarding = [MessageForwarding new];
    if ([messageForwarding respondsToSelector:anInvocation.selector]) {
        [anInvocation invokeWithTarget:messageForwarding];
    }
}

```

### 三种方法的选择

Runtime提供三种方式来将原来的方法实现代替掉，那该怎样选择它们呢？

- Method Resolution：由于Method Resolution不能像消息转发那样可以交给其他对象来处理，所以只适用于在原来的类中代替掉。

- Fast Forwarding：它可以将消息处理转发给其他对象，使用范围更广，不只是限于原来的对象。

- Normal Forwarding：它跟Fast Forwarding一样可以消息转发，但它能通过NSInvocation对象获取更多消息发送的信息，例如：target、selector、arguments和返回值等信息。


## Associated Objects

当使用Category对某个类进行扩展时，有时需要存储属性，Category是不支持的，这时需要使用Associated Objects来给已存在的类Category添加自定义的属性。Associated Objects提供三个API来向对象添加、获取和删除关联值：

- void objc\_setAssociatedObject (id object, const void \*key, id value, objc\_AssociationPolicy policy )

- id objc\_getAssociatedObject (id object, const void \*key )

- void objc\_removeAssociatedObjects (id object )

其中objc\_AssociationPolicy是个枚举类型，它可以指定Objc内存管理的引用计数机制。

``` OC
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,           /**< Specifies a weak reference to the associated object. */
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, /**< Specifies a strong reference to the associated object. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,   /**< Specifies that the associated object is copied. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_RETAIN = 01401,       /**< Specifies a strong reference to the associated object.
                                            *   The association is made atomically. */
    OBJC_ASSOCIATION_COPY = 01403          /**< Specifies that the associated object is copied.
                                            *   The association is made atomically. */
};
```

下面有个关于NSObject+AssociatedObject Category添加属性associatedObject的示例代码：

**NSObject+AssociatedObject.h**

``` OC
@interface NSObject (AssociatedObject)
@property (strong, nonatomic) id associatedObject;
@end
```

**NSObject+AssociatedObject.m**

``` OC
@implementation NSObject (AssociatedObject)
- (void)setAssociatedObject:(id)associatedObject
{
    objc_setAssociatedObject(self, @selector(associatedObject), associatedObject, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
- (id)associatedObject
{
    return objc_getAssociatedObject(self, _cmd);
}
@end
```

Associated Objects的key要求是唯一并且是常量，而SEL是满足这个要求的，所以上面的采用隐藏参数_cmd作为key。

## 总结
本文主要参考了 [https://www.csdn.net/article/2015-07-06/2825133-objective-c-runtime/1][3]

### 扩展阅读

- [玉令天下博客的Objective-C Runtime][4]
- [顾鹏博客的Objective-C Runtime][5]
- [Associated Objects][6]
- [Method Swizzling][7]
- [Method Swizzling和AOP实践][8]
- [Objective-C Runtime Reference][9]
- [What are the Dangers of Method Swizzling in Objective C?][10]
- [iOS程序员6级考试（答案和解释）][11]


[1]:https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/Articles/KVOImplementation.html

[2]: https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1

[3]: https://www.csdn.net/article/2015-07-06/2825133-objective-c-runtime/1

[4]: http://yulingtianxia.com/blog/2014/11/05/objective-c-runtime/

[5]: http://tech.glowing.com/cn/objective-c-runtime/

[6]: https://nshipster.com/associated-objects/

[7]: https://nshipster.com/method-swizzling/

[8]: http://tech.glowing.com/cn/method-swizzling-aop/

[9]: https://developer.apple.com/reference/objectivec/1657527-objective_c_runtime#//apple_ref/doc/uid/TP40001418

[10]: https://stackoverflow.com/questions/5339276/what-are-the-dangers-of-method-swizzling-in-objective-c

[11]: http://blog.sunnyxx.com/2014/03/06/ios_exam_0_key/

























