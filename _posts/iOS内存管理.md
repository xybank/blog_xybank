---
title: iOS内存管理
date: 2018-06-28
tags: [iOS]
categories: 卓平
description: iOS内存管理
---

本文是对《iOS与OS X多线程和内存管理》一书中内存管理一章知识的总结，文中大量引用了书中的观点和例子。


从学习iOS开始时iOS已经进入了ARC(自动引用计数)的时代了，内存无需手动管理，因此以前的学习中对于MRC关注甚少。但是随着平时在开发中碰到的问题越来越多的涉及到内存管理的知识，因此从MRC开始了解一些内存管理的知识是很有必要的。


## 思考MRC的四个要点
- 1.自己生成的对象，自己所持有
- 2.非自己生成的对象，自己也能持有
- 3.不在需要自己持有的对象时释放
- 4.非自己持有的对象无法释放


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
	[obj autorelease];
	return obj;
}

id obj1 = [obj0 object];//取得对象存在，但自己不持有对象
```
代码中调用了autorelease方法，用该方法使得取得的对象存在，但自己不持有对象。autorelease可以使得对象在超出指定生存范围时能够自动并正确地释放(调用release方法)。

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

PS: NSAutoreleasePool可以嵌套使用，嵌套使用时最内侧的NSAutoreleasePool对象最先释放。

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


## ARC规则  
### 所有权修饰符
要理解ARC，首先要理解ARC中追加的所有权声明。  

- __strong 修饰符
- __weak修饰符
- _____unsafe____unretained修饰符
- __autoreleaseing修饰符  


属性声明的属性			|	所有权修饰符
:--- 				 	| 	:---	
assign				  	|	_____unsafe____unretained修饰符
copy					|	__strong修饰符（被赋值的是复制的对象）
retain					|	__strong修饰符 
strong					|	__strong修饰符 
unsafe_unretained	|	_____unsafe____unretained修饰符
weak					|	__weak修饰符


#### __strong修饰符

__strong修饰符是id类型和对象类型默认的所有权修饰符。  

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

为了释放生成并持有的对象，MRC环境下对象调用的release方法。然而添加了__strong修饰符的对象在超出其变量作用域是，即在该变量被废弃时，会释放其被复制的对象。  

__strong修饰符标识对对象的强应用。持有强引用的变量在超出其作用域时被废弃，随着强引用的实效，引用的对象会随之释放。

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

综上可以看出，__strong修饰符的变量，不仅只在变量作用域中，在赋值上也能够正确地管理其对象的所有者。  
对于方法的参数也一样可以使用__strong修饰符。

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

通过__strong修饰符，不必再键入retain或者release，完美地满足了“引用计数式内存管理的思考方式”：

- 自己生成的对象，自己所持有
- 非自己生成的对象，自己也能持有
- 不在需要自己持有的对象时释放
- 非自己持有的对象无法释放 
前两项“自己生成的对象，自己所持有”和“非自己生成的对象，自己也能持有”只需通过带__strong修饰符的变量赋值便可达成。通过废弃带__strong修饰符的变量(变量作用域结束或者是成员变量所属对象废弃)或者对变量赋值，都可以做到“不需要自己持有的对象时释放”。最后一项“非自己持有的对象无法释放”，由于不能键入release，所以原本就无法执行。



#### __weak修饰符  

_____weak修饰符的一个重要作用就是解决_____strong修饰符所带来的“循环引用”问题。  
__weak修饰符提供弱引用。弱引用不能持有对象实例。以下代码就能解决上述代码循环引用

```OC
@interface Test:NSObject
{
	id __weak obj_;
}
- (void)setObject:(id __strong)obj;
@end
```
__weak修饰符有着一个特性，在持有某对象的弱引用时，若该对象被废弃，则此弱引用将自动失效且处于nil被赋值的状态。



#### _____unsafe____unretained修饰符
该修饰符正如其名，是一个不安全的修饰符。尽管ARC式的内存管理是编译器的工作，但附有_____unsafe____unretained修饰符的变量不属于编译器的内存管理对象。

附有_____unsafe______unretained修饰符的变量与__weak修饰符的变量一样，因为自己生成并持有的对象不能继续为自己所有，所以生成的对象会立即释放。**他们的差异在于弱引用持有的对象，在对象被废弃时weak变量会被nil赋值，而_____unsafe____unretained变量不会发生变化,所以在使用时要确保被修饰的变量的对象存在，负责会造成程序崩溃**



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

ARC下通过__autoreleaseing修饰符替代了autorelease方法。
然而显式地附加 __autoreleaseing修饰符同显式地附加 __strong修饰符一样罕见。
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

__weak修饰符只持有对象的弱引用，在访问过程中，该对象可能被废弃。只有把它注册到autoreleasepool中，在@autoreleasepool块结束之前才能确保对象一直存在。


最后一个可非显式地使用 __autoreleasing 修饰符的例子 

```OC
id obj = nil;
//等同于
id __strong obj = nil;


id *obj = nil;
//等同于
id __autoreleasing *obj = ni;


NSObject **obj = nil;
//等同于
NSObject *__autoreleasing* obj = nil;

```
























