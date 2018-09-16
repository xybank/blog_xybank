---
title: iOS内存管理补充
date: 2018-09-16
tags: [iOS]
categories: 卓平
description: 本文主要描述iOS内存管理的基础知识
---

## 思考MRC的四个要点
- 1.自己生成的对象，自己所持有
- 2.非自己生成的对象，自己也能持有
- 3.不在需要自己持有的对象时释放
- 4.非自己持有的对象无法释放

> PS: Objective-C内存管理方法中，实际上不包括在该语言中，而是包含在Cocoa框架中用于OS X、iOS应用开发。Cocoa框架中Foundation框架类库的NSObject类担负内存管理的职责。

#### 1.自己生成的对象，自己持有
只有以下名称开头的方法名才表示自己生成的对象自己持有。

- alloc
- new 
- copy
- mutableCopy

例如：

√：可以生成自己持有的对象的方法名  
×：不能生成自己持有的对象的方法名  

√					|	×
--- 				| 	---	
allocMyObject		|	allocate
newThatObject		|	newer
copyThis			|	copying
mutableCopyObj	|	mutableCopyed  
可以看出必须使用驼峰拼写法来命名

##### 深复制/浅复制
由于copy与mutableCopy涉及到iOS的深浅复制的问题，这里有必要一提。

问：**什么是深浅复制？**  
答：简单来说，浅复制就是两个变量指向了同一块内存区域，深复制就是两个变量指向了不同的内存区域，但是两个内存区域里面的内容是一样的。
    
**深浅复制的三个要点**  

- 可变对象的copy和mutableCopy方法都是深复制
- 不可变对象的copy方法是浅复制，mutableCopy方法是深复制
- copy方法返回的对象是不可变对象

上述三个要点对于非集合/集合类型来说都是适用的，
**但是对于集合类型的深复制来说只是进行了单层深复制，对于集合类型中的元素来说是浅复制**  

**集合对象的完全复制**  

- 使用 initWith***: copyItems:YES  方法。  
- 先将集合进行归档，然后再解档。
  
  
#### 2.非自己生成的对象，自己也能持有
用alloc/new/copy/mutableCopy以外的方法取得对象，因为非自己生成并持有，所以自己不是对象的持有者。 
 
``` OC  

//取得非自己生成并持有的对象，取得的对象存在但自己不持有对象。
id obj = [NSMutableArray array]; 

[obj retain]; //自己持有对象

```
通过retain方法，非自己生成的对象跟用alloc/new/copy/mutableCopy方法生成并持有的对象一样，成为了自己所持有的。  

#### 3.不再需要自己持有的对象时释放
自己持有的对象，一旦不再需要，持有者有义务释放该对象。释放使用release方法

``` OC  

id obj = [[NSObject alloc]init]; 

[obj release]; //释放对象
//指向对象的指针仍然被保留在变量obj中，但对象一经释放绝对不可访问。
```


``` OC  

//取得非自己生成并持有的对象，取得的对象存在但自己不持有对象。
id obj = [NSMutableArray array]; 

[obj retain]; //自己持有对象

[obj release];

```


**在某个方法中生成对象，并将其返还给该方法的调用方**

``` OC 
- (id)allocObject {
	id obj = [[NSObject alloc]init];
	return obj;
}

//使用allocObject意味着自己生成并持有
id obj1 = [obj0 allocObject];

```
直接返回alloc方法生成并持有的对象，就能让调用方也持有该对象。allocObject这个名称符合前文命名规则的。

**调用方法使得对象存在，但自己不持有对象**

``` OC
//方法命名不能以alloc/new/copy/mutableCopy开头的方法名
- (id)object {
	id obj = [[NSObject alloc]init];
	[obj autorelease];//该方法使取得的对象存在，但自己不持有对象
	return obj;
}

id obj1 = [obj0 object];//取得对象存在，但自己不持有对象
[obj1 retain];//自己持有对象
```
代码中调用了autorelease方法，用该方法使得取得的对象存在，但自己不持有对象。**autorelease可以使得对象在超出指定生存范围时能够自动并正确地释放(调用release方法)**。

根据命名规则，这些用来取得谁都不持有的对象的方法名不能以alloc/new/copy/mutableCopy开头

#### 4.无法释放非自己持有的对象
用alloc/new/copy/mutableCopy方法生成并持有的对象，或是用retain方法持有的对象，由于持有者是自己，所以在不需要该对象时需要将其释放。除此以外所有的对象不能释放。

``` OC
//例1
id obj = [[NSObject alloc]init];
[obj release];//正常释放
[obj release];//再释放崩溃

//例2
id obj1 = [obj0 object];
[obj1 release];//崩溃
```
 
### autorelease
autorelease的用处就是延迟对象释放的时机。  
像C语言的自动变量一样，当变量超出其作用域，变量就会自动废弃。注册到autoreleasepool中的对象当超出其作用域时，对象实例的release方法就会被调用。

**autorelease的具体使用方法**  
(1)生成并持有NSAutoreleasePool对象  
(2)调用已分配对象的autorelease实例方法  
(3)废弃NSAutoreleasePool对象

**PS:**

1. NSAutoreleasePool可以嵌套使用，嵌套使用时最内侧的NSAutoreleasePool对象最先释放。
2. 不能对NSAutoreleasePool调用autorelease方法，因为无论调用哪一个对象的autorelease实例方法，实现上是调用的NSObject类的autorelease实例方法。但是对于NSAutoreleasePool类，autorelease实例方法已被该类重载，因此运行时就会出错。

**autoreleasepool释放对象的时机**  
1.程序主循环的NSRunLoop一次迭代结束时。  
2.手动添加NSAutoreleasePool对象调用drain方法时。

``` OC
//使用方法1  只能在MRC下使用
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
[[[NSObject alloc]init]autorelease]
[pool drain];

//使用方法2   ARC和MRC下都能使用
@autoreleasepool {
	//MRC下时
	[[[NSObject alloc]init]autorelease]
}

```

综上可以看出因此应用程序不一定非得使用NSAutoreleasePool对象来进行开发工作。

**autoreleasepool的应用场景**   
对于可能大量产生autorelease的对象的情况，可以在合适的地方生成、持有或废弃NSAutoreleasePool对象。
  
``` OC
//典例：读入大量图像同时改变尺寸，会生成大量的对象。
for (int i=0; i<图像数; i++) {
	NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
	
	//读入图像大量产生autorelease的对象
	
	[pool drain];
} 
```

另外，可以通过NSAutoreleasePool类中的调试用非公开类方法showPools来确认已被autorelease的对象的情况。showPools会将现在的NSAutoreleasePool的状况输出到控制台。

``` OC
//该方法只能在iOS中使用
[NSAutoreleasePool showPools];

//在ARC中使用调用非公开函数
//声明
extern void _objc_autoreleasePoolPrint(void);
//输出
_objc_autoreleasePoolPrint();

```






## ARC规则  
### 所有权修饰符
要理解ARC，首先要理解ARC中追加的所有权声明。  

- \_\_strong修饰符
- \_\_weak修饰符
- \_\_unsafe\_\_unretained修饰符
- \_\_autoreleaseing修饰符  


属性声明的属性			|	所有权修饰符
:--- 				 	| 	:---	
assign				  	|	\_\_unsafe\_\_unretained修饰符
copy					|	\_\_strong修饰符（被赋值的是复制的对象）
retain					|	\_\_strong修饰符 
strong					|	\_\_strong修饰符 
unsafe\_unretained	|	\_\_unsafe\_\_unretained修饰符
weak					|	\_\_weak修饰符


#### \_\_strong修饰符

\_\_strong修饰符是**id类型和对象类型**默认的所有权修饰符。  

``` OC
//以下两行代码是完全等价的
id obj = [[NSObject alloc] init];
id __strong obj = [[NSObject alloc] init];

```

**那么__strong修饰符的作用又是什么呢？**

``` OC
/**
 * 以下代码的效果相同
 */
//ARC环境下
{
	id __strong obj = [[NSObject alloc] init];
	
	id __strong obj1 = [NSMutbaleArray array];
}

//MRC环境下
{
	id obj = [[NSObject alloc] init];
	[obj release];
	
	id obj1 = [NSMutbaleArray array];
}
```

为了释放生成并持有的对象，MRC环境下对象调用的release方法。然而添加了\_\_strong修饰符的对象在超出其变量作用域时，即在该变量被废弃时，会释放其被复制的对象。  

\_\_strong修饰符标识对对象的强应用。持有强引用的变量在超出其作用域时被废弃，随着强引用的失效，引用的对象会随之释放。

在取得非自己生成并持有的对象时，由于变量obj1为强引用，obj1也会持有对象，当超出作用域时，强引用失效，释放持有对象。

``` OC
id __strong obj0 = [[NSObject alloc]init];//对象A
id __strong obj1 = [[NSObject alloc]init];//对象B
id __strong obj2 = nil;

//obj0持有由obj1赋值的对象B的强引用，由于obj0被赋值，  
所以原先持有对象A的强引用失效，对象A的所有者不存在，因此废弃对象A
obj0 = obj1;  //obj0和obj1此时共同持有对象B

//obj2持有由obj0赋值的对象B的强引用
obj2 = obj0;	//此时对象B的持有者obj0、obj1和obj2
obj1 = nil;	//obj1对对象B的强引用失效
obj0 = nil;	//obj0对对象B的强引用失效
obj2 = nil;	//obj2对对象B的强引用失效

```

综上可以看出，\_\_strong修饰符的变量，不仅只在变量作用域中，在赋值上也能够正确地管理其对象的所有者。  
对于类的成员变量和方法的参数也一样可以使用\_\_strong修饰符。

``` OC
@interface Test:NSObject
{
	id __strong obj_;
}
- (void)setObject:(id __strong)obj;
@end


@implementation Test 
- (id)init 
{
	self = [super init];
	return self;
}

- (void)setObject:(id __strong)obj
{
	obj_ = obj;
}
@end



{
	//test持有Test对象的强引用
	id __strong test = [[Test alloc]init];
	
	//Test对象的obj_成员持有NSObject对象的强应用
	[test setObject:[[NSObject alloc]init]];
}
//test变量超出作用域，强引用失效，Test对象自动释放
//Test对象释放的同时，其中的obj_成员也被是释放。
//NSObject对象强引用失效，NSObject对象自动释放。

```

另外，\_\_strong修饰符合\_\_weak修饰符和\_\_autoreleaseing修饰符一起，可以保证附有这些修饰符的自动变量初始化为nil。

``` OC
id __strong obj0;
id __weak obj1;
id __autoreleasing obj2;

//以下源代码与上相同
id __storng obj0 = nil;
id __weak obj1 = nil;
id __autorelease obj2 = nil;
```

通过\_\_strong修饰符，不必再键入retain或者release，完美地满足了“引用计数式内存管理的思考方式”：

- 自己生成的对象，自己所持有
- 非自己生成的对象，自己也能持有
- 不在需要自己持有的对象时释放
- 非自己持有的对象无法释放 
前两项“自己生成的对象，自己所持有”和“非自己生成的对象，自己也能持有”只需通过带\_\_strong修饰符的变量赋值便可达成。通过废弃带\_\_strong修饰符的变量(变量作用域结束或者是成员变量所属对象废弃)或者对变量赋值，都可以做到“不需要自己持有的对象时释放”。最后一项“非自己持有的对象无法释放”，由于不能键入release，所以原本就无法执行。



#### __weak修饰符  

\_\_weak修饰符的一个重要作用就是解决\_\_strong修饰符所带来的“循环引用”问题。  
\_\_weak修饰符提供弱引用。弱引用不能持有对象实例。以下代码就能解决上述代码循环引用

```OC
@interface Test:NSObject
{
	id __weak obj_;
}
- (void)setObject:(id __strong)obj;
@end
```
\_\_weak修饰符有着一个特性，在持有某对象的弱引用时，若该对象被废弃，则此弱引用将自动失效且处于nil被赋值的状态。

\_\_weak修饰符只能用于iOS5以上及OS X Lion以上版本的应用程序。在iOS4以及OS X Snow Leopard的应用程序中可使用\_\_unsafe\_unretained修饰符来代替


#### \_\_unsafe\_unretained修饰符
该修饰符正如其名，是一个不安全的修饰符。**尽管ARC式的内存管理是编译器的工作，但附有\_\_unsafe\_unretained修饰符的变量不属于编译器的内存管理对象。**

附有\_\_unsafe\_unretained修饰符的变量与\_\_weak修饰符的变量一样，因为自己生成并持有的对象不能继续为自己所有，所以生成的对象会立即释放。**他们的差异在于弱引用持有的对象，在对象被废弃时weak变量会被nil赋值，而\_\_unsafe\_unretained变量不会发生变化,所以在使用时要确保被修饰的变量的对象存在，否则会造成程序崩溃**



#### autoreleasing修饰符

ARC下不能使用NSAutoreleasePool类。

```OC
//MRC下
NSAutoreleasePool *pool = [[NSAutorelease alloc]init];
id obj = [[NSObject alloc]init];
[obj autorelease];
[pool drain];

//ARC下
@autoreleasepool {
	id __autoreleaseing obj = [[NSObject alloc]init];
}

```

ARC下通过\_\_autoreleaseing修饰符替代了autorelease方法。
然而显式地附加 \_\_autoreleaseing修饰符同显式地附加 \_\_strong修饰符一样罕见。
这是为什么呢？看以下例子便能知道。

```OC
@autoreleasepool{
	id obj = [NSMutableArray array];
	//变量obj为强引用，对象本身也被注册到autoreleasepool
}
//超出作用域obj强引用失效，自动释放自己持有的对象。
//同时随着@autoreleasepool块的结束，注册到池中的所有对象都被自动释放了

```

在访问附有 __weak 修饰符的变量时，实际上必定要访问注册到autoreleasepool的对象。

```OC
id __weak obj1 = obj0;
NSLog(@"class=%@",[obj1 class]);

//以下源代码与此相同
id __weak obj1 = obj0;
id __autoreleasing tmp = obj1;
NSLog(@"class=%@",[tmp class]);

```

**为什么在访问附有 __weak 修饰符的变量时必须访问注册到autoreleasepool的对象呢？**

\_\_weak修饰符只持有对象的弱引用，在访问过程中，该对象可能被废弃。只有把它注册到autoreleasepool中，在@autoreleasepool块结束之前才能确保对象一直存在。


可非显式地使用\_\_autoreleasing 修饰符的例子 

```OC

id *obj = nil;
//等同于
id __autoreleasing *obj = ni;


NSObject **obj = nil;
//等同于
NSObject *__autoreleasing* obj = nil;

```


### 规则
在**ARC有效**的情况下编译源码，必须遵守一定的规则。具体规则如下

- **不能使用 retain/release/retainCount/autorelease**
- **不能使用 NSAllocateObject/NSDeallocateObject**
- **须遵守内存管理的方法命名规则**
- **不要显示调用dealloc**
- **使用@autorelease块替代NSAutoreleasePool**
- **不能使用区域（NSZone）**
- **对象型变量不能最为C语言结构体（struc/union）的成员**
- **显式转换“id”和“void \*”**

--
#### · 须遵守内存管理的方法命名规则
- alloc
- new
- copy
- mutableCopy

以上述名称开始的方法在返回对象时必须返回调用方所应当持有的对象。这点ARC与MRC是一致的。

只是ARC有效时要追加一条命名规则。  

- init  
以init开始的方法的规则会比alloc/new/copy/mutableCopy更严格。该方法必须是实例方法，并且需要返回对象。返回的对象应为对应的id类型
或该方法声明的对应类型，或者是该类的超类或子类。该返回对象并不会注册到autoreleasepool上。基本上只对alloc方法返回值的对象进行初始化处理并返回对象。

``` OC

//1. √ 该方法声明符合命名规则
- (id) initWithObject:(id)obj;

//2. × 不符合符合命名规则
- (void) initThisObject;

//3. × 不符合符合命名规则
- (void) initialize;
 
```

#### · 不显示调用dealloc
无论ARC还是MRC，只要对象的所有者都不持有该对象，该对象就被废弃。对象被废弃时，都会调用对对象的dealloc方法。

``` OC
//ARC 环境下
- (void)dealloc {
	//要执行的代码
	//[super dealloc]; ARC下会遵循无法显示调用dealloc这一规则，ARC会自动对此进行处理，因此不必写出。
}

//MRC 环境下
- (void)dealloc {
	//要执行的代码
	[super dealloc];
}
```

#### · 不能使用区域（NSZone）
ARC有效时，不能使用区域（NSZone）。
不管ARC是否有效，区域在现在的运行时系统中已单纯地被忽略。


#### · 对象型变量不能作为C语言结构体的成员
C语言的结构体（struct或union）成员中，如果存在Objective-C对象变量，便会引起编译错误。

``` OC
//×：ARC不允许结构体和联合中包含OC对象
struct Data {
	NSMutableArray *array;
};

```

**要把对象型的变量加入到结构体成员中时，可强制转换未void \*或是\_\_unsafe_unretained修饰符。附有\_\_unsafe\_unretained修饰符的变量不属于编译器的内存管理对象。如果管理时不注意复制对象的所有者，便有可能导致内存泄漏或崩溃。**

``` OC
struct Data {
	NSMutableArray __unsafe_unretained *array;
};
```

#### ·显式转换id和void *
MRC下，以下代码将id变量强制转换void *变量并不会出问题。

``` OC
//MRC环境
id obj = [[NSObject alloc] init];
void *p = obj;

id o = p;
[o release];

```

ARC下，上述代码会引起编译错误。
id型或对象型变量赋值给void *或者逆向赋值时都需要进行特定的转换。如果只想单纯的赋值，则可以使用“\_\_bridge”转换。

``` OC
id obj = [[NSObject alloc]init];
void *p = (__bridge void *)obj;
id o = (__bridge id)p;
```

**通过\_\_bridge转换的void \*和id，其安全性与赋值给\_\_unsafe_unretained修饰符相近，甚至会更低。**如果管理时不注意赋值对象的所有者，就会因悬垂指针而导致程序崩溃。


**\_\_bridge转换还有另外两种**  

- \_\_bridge\_retained   

\_\_bridge\_retained转换可使要转换赋值的变量也持有所赋值的对象。

``` OC
//ARC
id obj = [[NSObject alloc]init];
void *p = (__bridge_retained void *)obj;

//MRC
id obj = [[NSObject alloc]init];
void *p = obj;
[(id)p retain];
```


- \_\_bridge\_transfer

\_\_bridge\_transfer转换提供与此相反的动作，被转换的变量所持有的对象在该变量被赋值给转换目标之后随之释放。
``` OC
//ARC
id obj = (__bridge_transfer id)p;

//MRC
id obj = (id)p;
[obj retain];
[(id)p release];
```

如果使用上述两种转换，那么不使用id型或对象型变量也可以生成、持有以及释放对象，但这在ARC中并不推荐。

``` OC
//ARC
//生成并持有对象
void *p = (__bridge_retained void *)[[NSObject alloc]init];
//释放对象
(void)(__bridge_transfer id)p;

//以上代码与MRC下等价

//MRC
id p = [[NSObject alloc]init];
[p release];
```
















