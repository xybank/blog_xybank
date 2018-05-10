---
title: Java中的四种引用
date: 2018-05-09 10:00:00
tags: [Java]
categories: 蔡向炜
---

Java 中有四种引用：强引用、软引用、弱引用、虚引用。引用的存在，就是为了让开发者可以更好地管理内存。

<!--more-->

### 1. 强引用

**强引用是指创建一个对象并把这个对象赋给一个引用变量。** 当一个对象有具体指向强引用时，JVM 宁可抛出 OutOfMemory 也不会去回收它。强引用是我们平时最常用到的引用，如：

``` java
Person person = new Person();
```

``` java
String str = "Hello World !";
```

当然，Java 对象有自己的作用域，在作用域之外，对象已经相当于不存在了，对应的也会被回收。

如果希望回收强引用指向的对象，可以直接给引用赋值为空，即 `person = null;` ，那么 JVM 会在适当的时机回收该对象。例如，Vector 类的 clear 方法中就是通过将引用赋值为 null 来实现清理工作的。

### 2. 软引用（SoftReference）

**如果一个对象具有软引用：当内存足够时，不会被回收；当内存不足时，会被回收。** 

改变 eclipse JVM 内存（ Window -> Preferences -> Java -> Installs JREs -> Edit -> Default VM arguments 内容改为 `-Xmx65m -Xms64m -Xmn32m -Xss16m` ，其中 `-Xmx65m` 为最大内存 ），做如下测试：

``` java
int COUNT = 8777180; // 根据 JVM 内存调整
SoftReference<Person> personSR = new SoftReference<Person>(new Person());
System.gc();
System.out.println("1:" + personSR.get());

Person[] persons = new Person[COUNT];

System.gc();
System.out.println("2:" + personSR.get());
```

输出如下：

``` log
1:com.laocaixw.test.reference.Person@15db9742
2:null
```

可以看到 persons 达到一定大小时，软引用 personSR 持有的 person 会被回收，所以 `personSR.get()` 获取到的值为 null 。

**软引用可以用来实现内存敏感的高速缓存，比如网页缓存、图片缓存等。** 使用软引用能防止内存泄露，增强程序的健壮性。

另外，需要注意的是：SoftReference 是用来管理引用对象的，但是 SoftReference 本身也是一个对象。当 SoftReference 所管理的对象被回收后，SoftReference 本身并没有被回收，但是它已经没有任何作用了。如果内存中存在大量无用的 SoftReference，也会导致内存泄漏。所以，我们应该在合适的时机将无用的 SoftReference 赋值为 null，释放掉它所占用的内存。

SoftReference 可以和 ReferenceQueue 一起使用。创建 SoftReference 的时候将 ReferenceQueue 传入 SoftReference 的构造方法。当 SoftReference 所管理的对象被回收的时候，SoftReference 就会被放到 ReferenceQueue 中。

``` java
int COUNT = 8777180;
ReferenceQueue<Person> queue = new  ReferenceQueue<Person>();
SoftReference<Person> personSR = new SoftReference<Person>(new Person(),queue);
System.gc();
System.out.println("1:" + personSR.get());
System.out.println("1:" + personSR);
System.out.println("1:" + queue.poll());

Person[] persons = new Person[COUNT];

System.gc();
System.out.println("2:" + personSR.get());
System.out.println("2:" + queue.poll());
```

输出如下：

``` log
1:com.laocaixw.test.reference.Person@15db9742
1:java.lang.ref.SoftReference@6d06d69c
1:null
2:null
2:java.lang.ref.SoftReference@6d06d69c
```

### 3. 弱引用（WeakReference）

弱引用相对软引用，生命周期更短暂。**当 JVM 进行垃圾回收时，无论内存是否充足，弱引用管理的对象都会被回收。** 

``` java
WeakReference<Person> personWR = new WeakReference<Person>(new Person());
System.out.println("1:" + personWR.get());
System.gc();
System.out.println("2:" + personWR.get());
```

输出如下：

``` log
1:com.laocaixw.test.reference.Person@15db9742
2:null
```

当然，WeakReference 和 SoftReference 一样，也可以和 ReferenceQueue 一起使用。

### 4. 虚引用（PhantomReference）

虚引用也称幽灵引用，它和前面的软引用、弱引用不同，并不影响对象的生命周期。如果一个对象与虚引用关联，则跟没有引用与之关联一样，在任何时候都可能被垃圾回收器回收。

``` java
ReferenceQueue<Person> queue = new ReferenceQueue<Person>();
Person person = new Person();
PhantomReference<Person> personPR = new PhantomReference<Person>(person, queue);
System.out.println("1:" + personPR.get());
System.out.println("1:" + queue.poll());

System.gc();
System.out.println("2:" + personPR.get());
System.out.println("2:" + queue.poll());

person = null;
System.gc();
System.out.println("3:" + personPR.get());
System.out.println("3:" + queue.poll());
```

输出如下：

``` log
1:null
1:null
2:null
2:null
3:null
3:java.lang.ref.PhantomReference@15db9742
```

PhantomReference 必须和 ReferenceQueue 一起使用。当与 PhantomReference 关联的对象被 JVM 回收时，这个 PhantomReference 就会被放到 ReferenceQueue 中。程序可以通过判断 ReferenceQueue 中是否加入了该 PhantomReference 来判断该对象是否被回收了。

