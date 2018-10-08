---
title: iOS GCD
date: 2018-09-28
tags: [iOS]
categories: 卓平
description: 本文主要描述iOS Grand Central Dispath(GCD)
---

本文主要参考了《Objective-C 高级编程》中的GCD一章的内容。

## 1. GCD 概要

### 1.1 什么是GCD

> Grand Central Dispath(GCD)是异步执行的技术之一。一般将应用程序中记述的线程管理用的代码在**系统级**中实现。开发者只需要定义想执行的任务并追加到适当的Dispatch Queue中，GCD就能生成必要的线程并计划执行任务。由于线程管理是作为系统的一部分来实现的，因此可统一管理，也可执行任务，这样就比以前的线程更有效率。————苹果官方说明

GCD简洁明了，实现了极为复杂繁琐的多线程编程。

``` OC
dispatch_asyn(queue, ^{
	/* 
	 * 长时间处理
	 * 如数据库访问等
	 */
	 
	 /*
	  * 长时间处理结束，主线程使用该处理结果
	  */
	  dispatch_async(dispatch_get_main_queue(),^{
	  		/*
	  		 * 只在主线程可执行的处理
	  		 * 例如用户界面更新
	  		 */
	  });
});
```

在导入GCD之前，Cocoa框架提供了NSObject类的performSelectorBackgound:withObject实例方法和performSelectorOnMainThread实例方法等简单的多线程编程技术。

``` OC
//NSObject performSelectorInBackgound:withObject:方法中执行后台线程
- (void)launchThreadByNSObject_performSelectorInBackground_withobject {
	[self performSelectorInBackgound:@selector(doWork) withObject:nil];
}

//后台线程处理方法
- (void)doWork {
	NSAutoreleasePool *pool = [[NSAutoreleasePool alloc]init];
	/* 
	 * 长时间处理
	 * 如数据库访问等
	 */
	 
	 /*
	  * 长时间处理结束，主线程使用该处理结果
	  */
	 [self performSelectorOnMainThread:@selector(doneWork) withObject:nil waitUntilDone:NO];
	 [pool drain];
}

//主线程处理方法
- (void)doneWork {
	
}

```
performSelector系方法确实比使用NSThread类进行多线程编程要简单，但是与GCD先比，结果一目了然，且通过GCD提供的系统级线程管理可提高执行效率。


### 1.2 多线程编程
源码通过编译器转换为CPU命令（二进制代码），应用启动后首先将包含在程序中的CPU命令配置到内存中。CPU从应用程序指定的地址开始，一个一个的执行命令列。由于一个CPU一次只能执行一个命令列，不能执行某处分开的并列的两个命令，因此通过CPU执行的CPU命令就好比一条无分叉的路径，其执行不会出现分歧。

这里说到的“一条无分叉的路径”即为线程。尽管有多核CPU，但是一个CPU一次只能执行一个命令列为一条无分叉的路径仍然不变。

无分叉的路径不止1条，存在有多条即为“多线程”。在多线程中，一个CPU执行多条不同路径上不同命令。

**1个CPU核一次能够执行的CPU命令始终为1。那么如何才能在多条路径中执行CPU命令列呢？**

在OS X和iOS的核心XNU内核在发生操作系统事件时（如每隔一定时间，唤起系统调用等情况）会切换执行路径。执行中路径的状态，例如CPU的寄存器等信息保存到各自专用的内存块中，从切换目标路径专用的内存中复原CPU寄存器等信息，继续执行切换路径的CPU命令列。这被称为“上下文切换”。

由于使用多线程的程序可以在某个线程和其他线程之间反复多次进行切换上下文，且速度极快。看上去就好像1个CPU核能够并列地执行多个程序一样。这称之为——“并发”。在多个CPU的情况下就不是看上去像了，而是真的提供多个CPU核并行的执行多个线程。

这种利用多线程编程的技术就被称为“多线程编程”。

多线程编程实际是一个容易发生各种问题的编程技术。比如资源竞争、死锁、使用太多线程会消耗大量内存等。

要回避这些问题有许多方法，但程序都会更加复杂。尽管极易发生问题，也应该多使用多线程编程。因为使用多线程编程可保证应用程序的响应性能。

## 2. GCD的API
### 2.1 Dispatch Queue

``` OC
dispatch_async(queue, ^{
	//想执行的任务
});
```
开发者要做的只是定义想执行的任务并追加到适当的Dispatch Queue中。
代码中使用Block语法来定义想要执行的任务，并通过dispatch\_async函数追加到赋值变量queue中。

Dispatch\_queue是执行处理的队列，队列按照先进先出的原则执行处理添加到队列中的任务。

队列分两种，一种是等待现在执行中处理的串行队列(DISPATCH\_QUEUE\_SERIAL)，另一种是不等待现在执行处理的并发队列(DISPATCH\_QUEUE\_CONCURRENT)。

比较这两种队列

``` OC
//串行队列
dispatch_async(queue, blk0);
dispatch_async(queue, blk1);
dispatch_async(queue, blk2);
dispatch_async(queue, blk3);
dispatch_async(queue, blk4);
dispatch_async(queue, blk5);
dispatch_async(queue, blk6);

//串行队列输出
blk0
blk1
blk2
blk3
blk4
blk5
blk6


//并发队列
dispatch_async(queue, blk0);
dispatch_async(queue, blk1);
dispatch_async(queue, blk2);
dispatch_async(queue, blk3);
dispatch_async(queue, blk4);
dispatch_async(queue, blk5);
dispatch_async(queue, blk6);

//并发队列输出，由于并发执行处理的数量取决于当前的系统的状态，所以输出的结果不是顺序的。
blk1
blk0
blk2
blk3
blk6
blk4
blk5

```
XNU内核决定应当使用的线程数，并只生成所需的线程执行处理，线程都由XNU内核来管理。Concurrent Dispatch Queue中执行处理时，执行顺序会根据处理内容和系统状态发生改变。它不同于执行顺序固定的Serial Dispatch Queue。在不能改变执行的处理顺序或不想并发执行多个处理时使用。

### 2.2 dispatch\_queue\_create

``` OC

/*	queue.h中的声明
 *	dispatch_queue_t
 *	dispatch_queue_create(const char *_Nullable label,dispatch_queue_attr_t _Nullable attr);
 */
  
dispatch_queue_t queue = dispatch_queue_create("com.example.gcd.myqueue",NULL);
```
**dispatch\_queue\_t**：队列类型  
**label**：线程标识符，用于标记线程  
**dispatch_queue_attr_t**：队列的属性，NULL、DISPATCH\_QUEUE\_SERIAL、DISPATCH\_QUEUE\_CONCURRENT等，其中NULL与DISPATCH\_QUEUE\_SERIAL等价。

虽然Serial Dispatch Queue 和 Concurrent Dispatch Queue受到系统资源的限制，但用dispatch\_queue\_create可以创建任意多个Dispatch Queue。

当生成多个Serial Dispatch Queue是，各个Serial Dispatch Queue将并发执行。但是**一旦生成Serial Dispatch Queue并追加处理，系统对于Serial Dispatch Queue就只生成并使用一个线程**。如果创建2000个Serial Dispatch Queue就会有2000个线程生成。这样就会消耗大量的内存，引起大量的上下文切换，大幅度降低系统的响应性能。
**只在为了避免多线程编程问题之一——多个线程更新相同资源导致数据竞争时使用Serial Dispatch Queue。**

当想并发执行不发生数据竞争等问题的处理时，使用Concurrent Dispatch Queue。而且**对于Concurrent Dispatch Queue来说，不管生成多少，由于XNU内核只使用有效管理的线程**，因此不会发生Serial Dispatch Queue的问题。

从iOS6.0起，GCD对象就被纳入ARC的管理范畴，ARC程序中不再需要调用dispatch_release来释放GCD对象。


### 2.3 Main Dispatch Queue/Global Dispatch Queue

系统为我们提供了Main Dispatch Queue和Global Dispatch Queue。

**Main Dispatch Queue：**

主线程中执行的Dispatch Queue。因为主线程只有一个，所以Main Dispatch Queue自然就是Serial Dispatch Queue。

追加到Main Dispatch Queue的处理在主线程RunLoop中执行。因此将将用户界面更新的一些必须放在主线程中执行的处理追加到Main Dispatch Queue使用。

**Global Dispatch Queue：**

**Global Dispatch Queue是所有应用程序都能够使用的Concurrent Dispatch Queue。**没必要通过dispatch\_queue\_create函数逐个生成Concurrent Dispatch Queue。只需获取Global Dispatch Queue使用即可。

Global Dispatch Queue有4个执行优先级

- 高优先级（Hight Priority）
- 默认优先级（Default Priority）
- 低优先级（Low Priority）
- 后台优先级（Background Priority）

优先顺序从高到低，通过XUN内核管理的用于Global Dispatch Queue的线程，将各自使用的Global Dispatch Queue的执行优先级作为线程的执行优先级使用。

但是通过XUN内核用于Global Dispatch Queue的线程并不能保证实时性，因此执行优先级只是大致的判断。

``` OC
//Main Dispath Queue的获取方法
dispatch_queue_t mainDispatchQueue = dispatch_get_main_queue();

//Global Dispatch Queue(高优先级)的获取方法
dispatch queue_t globalDispatchQueueHigh = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0);

//Global Dispatch Queue(默认优先级)的获取方法
dispatch queue_t globalDispatchQueueDefault = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

//Global Dispatch Queue(低先级)的获取方法
dispatch queue_t globalDispatchQueueLow = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0);

//Global Dispatch Queue(后台优先级)的获取方法
dispatch queue_t globalDispatchQueueHigh = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0);

```

### 2.4 dispatch\_set\_target\_queue

**用途：变更生成的Dispatch Queue的执行优先级**

**PS：**dispatch\_queue\_create函数生成的Dispatch Queue不管是Serial Dispatch Queue还是Concurrent Dispatch Queue,都是用与默认优先级Global Dispatch Queue相同执行优先级的线程。

``` OC
dispatch_queue_t mySerialDispatchQueue = dispatch_queue_create("com.example.gcd.mySerialDispatchQueue",NULL);

dispatch_queue_t globalDispatchQueueBackground = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0);

dispatch_set_target_queue(mySerialDispatchQueue, globalDispatchQueueBackground);

```
dispatch\_set\_target\_queue函数的第一个参数是要变更优先级的Dispatch Queue，第二个是指定要使用的执行优先级相同的目标Dispatch Queue。但是如果要变更的Dispatch Queue指定了系统提供的Main Dispatch Queue和Global Dispatch Queue则不知道会出现什么情况，因此这些均不可指定。

**用法：**将Dispatch Queue指定为dispatch\_set\_target\_queue的函数参数，不仅可以变更Dispatch Queue的执行优先级，还可以作为Dispatch Queue的执行阶层。如果在多个Serial Dispatch Queue中用dispatch\_set\_target\_queue函数指定目标为某一个Serial Dispatch Queue，那么原本应该并发执行的多个Serial Dispatch Queue，在目标Serial Dispatch Queue上只能同时执行一个处理

### 2.5 dispatch\_after

**用途：在指定时间追加处理到Dispatch Queue。**

``` OC
dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, 3ull*NSEC_PER_SEC);
dispatch_after(time, dispatch_get_main_queue(), ^{
	NSLog(@"waited at least three seconds.");
});
```

第一个参数指定时间用的dispatch_time_t类型值中指定的时间开始，到第二个参数指定的毫微秒单位时间后的时间。

dispatch\_after函数并不是在指定时间后执行处理，而只是在指定时间追加处理到Dispatch Queue。
因为Main Dispatch Queue在主线程的RunLoop中执行，所以在比如每隔1/60秒执行的RunLoop中，Block最快在3秒后执行，最慢在3秒+1/60秒后执行，并且Main Dispatch Queue有大量处理追加或主线程本身有延迟是，这个时间会更长。


### 2.6 Dispatch Group

**用途：监听和等待Group中所有执行处理的情况。**

``` OC
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_t group = dispatch_group_create();

dispatch_group_async(group, queue, ^{NSLog(@"blk0");});
dispatch_group_async(group, queue, ^{NSLog(@"blk1");});
dispatch_group_async(group, queue, ^{NSLog(@"blk2");});

dispatch_group_notify(group, dispatch_get_main_queue(),^{NSLog(@"done");});

//执行结果
/* blk1
 * blk2
 * blk0
 * done
 */
```

以上代码，追加处理的执行顺序不定。执行时会发生变化，但是done一定是在最后输出的。Dispatch_Goup监听这些处理执行结束，一旦检测到所有处理执行结束，就可将结束的处理追加到Dispatch Queue中。这就是使用Dispatch Gourp的原因。

另外，Dispatch Gourp中也可以使用dispatch\_group\_wait函数仅等待全部处理执行结束。

``` OC
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

dispatch_group_async(group, queue, ^{NSLog(@"blk0");});
dispatch_group_async(group, queue, ^{NSLog(@"blk1");});
dispatch_group_async(group, queue, ^{NSLog(@"blk2");});

dispatch_group_wait(group, DISPATCH_TIME_FOREVER);

```
函数的第二个参数表示等待时间，这里的DISPATCH_TIME_FOREVER意味着永久等待。只要Dispatch Group的处理尚未执行结束，就会一直等待，中途不能取消。

``` OC
dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, 1ull * NSEC_PER_SEC);
long result = dispatch_group_wait(group, time);
if (result == 0) {
	//属于Dispatch Group的全部处理执行结束
}
else {
	//属于Dispatch Group的某个处理还在执行中
}
```

dispatch\_group\_wait函数的返回值不为0则表示经过了指定时间，但是属于Dispatch Group的某一个处理还在执行中。如果返回值为0则表示全部处理执行结束。

指定DISPATCH\_TIME\_FOREVER，则返回值必定为0。

指定DISPATCH\_TIME\_NOW，则不用任何等待即可判定属于Dispatch Group的处理是否结束。
在主线程的RunLoop的每次循环中，可检查执行是否结束，从而不耗费多余的等待时间。但是一般多使用dispatch\_group\_notify函数来实现，因为它更简洁明了。

### 2.7 dispatch\_barrier\_async
**用途：可以避免多线程数据竞争引发的问题。**

``` OC
dispatch_queue_t queue = dispatch_queue_create("com.example.gcd",DISPATCH_QUEUE_CONCURRENT);

dispatch_async(queue, blk0_for_reading);
dispatch_async(queue, blk0_for_reading);
dispatch_async(queue, blk0_for_reading);
dispatch_async(queue, blk0_for_reading);
dispatch_async(queue, blk0_for_writing);
dispatch_async(queue, blk0_for_reading);
dispatch_async(queue, blk0_for_reading);

```

以上代码由于追加处理的执行顺序是随机的，那么在读写操作时就会出现问题。

但是通过dispatch\_barrier\_async就能解决这个问题

``` OC
dispatch_queue_t queue = dispatch_queue_create("com.example.gcd",DISPATCH_QUEUE_CONCURRENT);

dispatch_async(queue, blk0_for_reading);
dispatch_async(queue, blk0_for_reading);
dispatch_async(queue, blk0_for_reading);
dispatch_async(queue, blk0_for_reading);
dispatch_barrier_async(queue, blk0_for_writing);
dispatch_async(queue, blk0_for_reading);
dispatch_async(queue, blk0_for_reading);

```
当程序执行到dispatch\_barrier\_async时，就会等到队列中的其他处理全部结束后，再它的处理追加到队列中。

### 2.8 dispatch\_sync

dispatch\_sync，即是同步处理，意味着它会阻塞线程，直至执行的处理执行结束。可以说是简易版的dispatch\_group\_wait函数。

但是使用时要注意，可能会引起死锁的问题。

``` OC
dispatch_queue_t queue = dispatch_get_main_queue();
dispatch_sync(queue, ^{NSLog(@"Hello?");});
```
由于主线程被阻塞，导致被追加的处理需要等待被阻塞的线程先执行完处理。然而主线程正是要执行这个处理，因此就会产生死锁。

在Serial Dispatch Queue中也会引起相同的问题。

``` OC
dispatch_queue_t queue = dispatch_queue_create("com.example.gcd", NULL);
dispatch_async(queue, ^{
	dispatch_sync(queue, ^{NSLog(@"hello?");});
});

```

### 2.9 dispatch\_apply
**用途：dispatch\_apply函数是dispatch\_sync函数和Dispatch Group的关联API。该函数按指定次数将指定Block追加到指定的Dispatch Queue中，并等待全部处理直接结束**

``` OC
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_apply(10, queue, ^{size_t index}) {
	NSLog(@"%zu",index);
});
NSLog(@"done");

//输出结果
/*
 * 4
 * 1
 * 0
 * 2
 * 3
 * 5
 * 6
 * 7
 * 8
 * 9
 * done
 */
```
由于dispatch\_apply函数会阻塞线程，因此推荐在dispatch\_async函数中非同步地执行dispatch\_apply函数。

### 2.10 dispatch\_suspend/dispatch\_resume

**用途：dispatch\_suspend可将整个线程挂起，dispatch\_resume将挂起的线程重新启动。执行函数对已执行的处理没有影响。挂起后尚未执行的处理停止执行。而恢复则使得这些处理能够继续执行。**

### 2.11 dispatch\_once

**用途：保证应用程序执行中只执行一次指定处理。**

``` OC
//方法一
static int initialized = NO;
if (initialized == NO) {
	//初始化
	initialized = YES;
}

//方法二
static dispatch_onece_t pred;
dispatch_once(&pred, ^{
	//初始化
});
```

以上代码看似能达到相同的效果，但是通过dispatch_once函数，方法二即使在多线程环境下执行，也可以保证百分之百安全。

### 2.12 Dispatch Semaphore

**用途：更细粒度的避免多线程带来的数据竞争问题。**

``` OC
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
NSMutableArray *array = [[NSMutableArray alloc]init];
for (int i=0; i<100000; ++i) {
	dispatch_async(queue, ^{
		[array addObject:[NSNumber numberWithInt:i]];
	});
}
```
以上代码异步更新了NSMutableArray类的对象，所以执行后有内存错误导致应用崩溃的概率很高，此时应该使用Dispatch Semaphore。

Dispatch Semaphore是持有计数的信号量，该计数是多线程编程中的计数类型信号量。计数为0时等待，计数为1或大于1时，减去1而不等待。

``` OC
//初始化计数值为1的semaphore
dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);

//dispatch_semaphore_wait等待信号量的计数值大于或等于1。该函数会减去1并返回计数值，当值为0时，该函数可以安全的进行排他处理。
dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);

//当排他处理结束时，就可以通过dispatch_semaphore_signal将计数值加1
dispatch_semaphore_signal(semaphore);

```

实际中的应用代码实例

``` OC
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, O);

/*
 * 生成Dispatch Semaphore
 * Dispatch Semaphore的技术初始值设为1
 * 保证可访问的NSMutableArray类对象的线程
 * 同时只能有一个。
 */
 
dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);

NSMutableArray *array = [[NSMutableArray alloc]init];

for (int i=0; i<100000; ++i) {
	dispatch_async(queue, ^{
		/*
		 * 等待Dispatch Semaphore
		 * 
		 * 一直等待，直到信号量大于或等于1
		 */
		dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
		
		/*
		 * 此时信号量为0
		 * 
		 * 由于可以访问NSMutableArray类对象的线程
		 * 只有一个，因此可以安全的更新。
		 */
		[array addObject:[NSNumber numberWithInt:i]];
		
		/*
		 * 排他处理结束后，计数值加1
		 */
		dispatch_semaphore_signal(semaphore);
	});
}

```

























