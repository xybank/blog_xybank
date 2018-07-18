---
title: Rxjava2学习总结
date: 2018-07-18 16:00:00
tags: [Android,rxjava2,响应函数式,背压]
categories: 彭连
description: 主要就是对Rxjava2的学习总结
---

## 简述
对于Rxjava1与Rxjava2之前在项目中多少都使用过，由于Rxjava1与Rxjava2并无继承关系，后者在背压支持上更优秀且前者已经不再维护，所以着重介绍Rxjava2。

### 名词介绍
函数响应式编程是函数式编程和响应式编程这两大颠覆传统的牛逼编程范式叠加后的产物，编程界的牛逼二次方。
#### 函数式编程
是一种通过函数或者函数组合调用来处理数据，获取结果的编程范式。
#### 响应式编程
是一种面向数据流以及变化传播的一种范式，其中变化传播在程序中也是转换为数据流的形式进行处理。
#### 函数响应式编程
是一种通过一系列的函数组合调用来发射、转变以及监听，响应数据流的编程范式。在RxJava中，函数响应式编程具体表现为一个观察者（Observer）订阅一个可观察对象（Observable），通过创建可观察对象发射数据流，经过一系列操作符（Operators）加工处理和线程调度器（Scheduler）在不同线程间的转发，最后由观察者接受并做出响应的一个过程。

在RxJava2中，提供了五对观察者模式组合来完成这一系列的过程，每一对组合依靠其可调用的一系列函数的差异，而具有各自的特点。这五类组合（前为可观察对象后为对应的观察者）分别是：

```
第一组：ObservableSource/Observer
一次可发送单条数据或者数据序列onNext，可发送完成通知onComplete或异常通知onError，不支持背压。

第二组：Publisher/Subscriber
第一组基础上进行改进，支持背压，一次可发送单条数据或者数据序列onNext，可发送完成通知onComplete或异常通知onError，但效率没有第一组高。

第三组：SingleSource/SingleObserver
第一组简化版，只能发送单条数据onSuccess，或者异常通知onError

第四组：CompletableSource/CompletableObserve
第一组简化版，不能发送数据，只发送完成通知onComplete或者异常通知onError

第五组：MaybeSource/MaybeObserver
第三，第四组的合并版，只能发送单条数据onSuccess和完成通知onComplete或者发送一条异常通知onError
```

## Rxjava2使用介绍
先来一波代码：
```
//被观察者
        Observable observable = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> e) throws Exception {
                e.onNext("hello world")  //执行observer的onNext()回调方法
                e.onNext("hello world");
                e.onComplete();//执行onComplete()
            }
        });
        //观察者
        Observer observer = new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
              //Disposable是观察者Observer与可观察对象Observable建立订阅关系后生成的用来取消订阅关系和判断订阅关系是否存在的一个接口。
            }
            @Override
            public void onNext(String o) {
                //默认执行的方法,无调用限制
            }
            @Override
            public void onError(Throwable e) {
              //发送异常通知，只会执行一次与onComplete互斥,不在执行后面的操作
            }
            @Override
            public void onComplete() {
              //发送完成通知，只会执行一次与onError互斥,不在执行后面的操作
            }
        };
        //绑定形成订阅关系
        observable.subscribe(observer);
```
rxjava2与rxjava1一样都是基于观察者模式，所以会包含观察者与被观察者，被观察者（Observable/Flowable）通过发送事件告知（订阅的关系）观察者然后观察者进行消费操作，这就是最简单的一个流程。

以上注释都很清楚，就不多说了，需要说明的一点是如果调用Disposable的dispose()方法解除绑定，对于被观察者发送事件是没影响的，只不过Observer是不会再消费后面的事件了。

### 操作符
操作符（Operators）：其实质是函数式编程中的高阶函数，是对响应式编程的各个过程拆分封装后的产物，以便于我们操作数据流。

操作符大致可以分为一下几个类别，以下将对常用的几个操作符作介绍:
#### 创建操作符
```
//创建一个Observable，可接受一个或多个参数，将每个参数逐一发送
        Observable.just("hello world").subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                System.out.println(s);  //打印hello world
            }
        });
        //fromArray:创建一个Observable，接受一个数组，并将数组中的数据逐一发送
        //fromIterable：</b>创建一个Observable，接受一个可迭代对象，并将可迭代对象中的数据逐一发送
        //range：</b>创建一个Observable，发送一个范围内的整数序列
        //Observable.range(1,10);//从1开始以步长1递增发送10个数据
        Observable.range(0,5).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                System.out.println(integer); //依次打印0-5
            }
        });
```
#### 过滤操作符
```
 //filter：filter使用Predicate 函数接口传入条件值，来判断Observable发射的每一个值是否满足这个条件，如果满足，则继续向下传递，如果不满足，则过滤掉。
        Observable.range(0,10).filter(new Predicate<Integer>() {
            @Override
            public boolean test(Integer integer) throws Exception {
                return integer % 3 == 0;  //判断条件  为true则传递给Observer
            }
        }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                System.out.println(integer);  //打印0，3，6，9
            }
        });
        //distinct:过滤掉重复的数据项，过滤规则为：只允许还没有发射过的数据项通过。
        Observable.just(1,1,1,2,3,3,4,4,5).distinct().subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                System.out.println(integer);  //打印1，2，3，4，5
            }
        });
        //当然也可以将filter与distinct进行组合链式使用
        Observable.just(1,1,1,2,3,3,4,4,5,6).distinct().filter(new Predicate<Integer>() {
            @Override
            public boolean test(Integer integer) throws Exception {
                return integer % 3 == 0;
            }
        }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                System.out.println(integer);  //3，6
            }
        });
```

#### 变换操作符
```
 //map:对Observable发射的每一项数据应用一个函数，执行变换操作
        //map操作符，需要接收一个函数接口Function<T,R>的实例化对象，实现接口内R apply(T t)的方法，在此方法中可以对接收到的数据t进行变换后返回。
       Observable.range(0,5).map(new Function<Integer, String>() {
           @Override
           public String apply(Integer integer) throws Exception {
               return "item:"+integer * integer;
           }
       }).subscribe(new Consumer<String>() {
           @Override
           public void accept(String s) throws Exception {
               System.out.println(s); //打印item:0，item:1，item:4，item:9，item:16，item:25
           }
       });
       //flatMap:将一个发射数据的Observable变换为多个Observable，然后将多个Observable发射的数据合并到一个Observable中进行发射
        Integer[] num1 = new Integer[]{1,2,3,4};
        Integer[] num2 = new Integer[]{5,6};
        Integer[] num3 = new Integer[]{7,8,9};
        Observable.just(num1, num2, num3).flatMap(new Function<Integer[], Observable<Integer>>() {
            @Override
            public Observable<Integer> apply(Integer[] integers) throws Exception {
                return Observable.fromArray(integers);
            }
        }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                System.out.println(integer); //依次打印1-9
            }
        });
```
#### 组合操作符
```
 //mergeWith：合并多个Observable发射的数据，可能会让Observable发射的数据交错。
        Integer []num4 = new Integer[]{1,2,3,4,5};
        Observable.just(5,6,7,8).mergeWith(Observable.fromArray(num4)).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                System.out.println(integer);  //打印1,2,3,5,6,4,7,8  可能会发生错乱
            }
        });
        //concatWith：同mergeWith一样，用以合并多个Observable发射的数据，但是concatWith不会让Observable发射的数据交错。
        Integer []num5 = new Integer[]{1,2,3,4,5};
        Observable.just(5,6,7,8).concatWith(Observable.fromArray(num4)).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                System.out.println(integer);  //打印1,2,3,4,5,6,7,8 不会错乱
            }
        });
```

#### 聚合操作符
```
//zipWith：将多个Obversable发射的数据，通过一个函数BiFunction对对应位置的数据处理后放到一个新的Observable中发射，所发射的数据个数与最少的Observabel中的一样多。
       String []colors = new String[]{"黄色","红色","绿色","橙色","褐色","黑色"};
       Observable.just(1,2,3,4,5,6,7).zipWith(Observable.fromArray(colors), new BiFunction<Integer, String, String>() {
           @Override
           public String apply(Integer integer, String s) throws Exception {
               return integer + s;
           }
       }).subscribe(new Consumer<String>() {
           @Override
           public void accept(String s) throws Exception {
               System.out.println(s);  //打印1黄色 2红色 3绿色...6黑色
           }
       });
```

### Scheduler线程调度器
Scheduler(线程调度器)赋予RxJava简洁明了的异步操作,可以说是RxJava中最值得称道的地方。Scheduler(线程调度器)可以让RxJava的线程切换变得简单明了，即使程序逻辑变得十分复杂，他依然能够保持简单明了。

#### subscribeOn
```
Observable<T> subscribeOn(Scheduler scheduler) 
```
subscribeOn通过接收一个Scheduler参数，来指定对数据的处理运行在特定的线程调度器Scheduler上。若多次设定，则只有一次起作用。

#### observeOn
```
Observable<T> observeOn(Scheduler scheduler)
```
observeOn同样接收一个Scheduler参数，来指定下游操作运行在特定的线程调度器Scheduler上。若多次设定，每次均起作用。

#### Scheduler种类
```
Schedulers.io():用于IO密集型的操作，例如读写SD卡文件，查询数据库，访问网络.
Schedulers.newThread()：在每执行一个任务时创建一个新的线程，不具有线程缓存机制，因为创建一个新的线程比复用一个线程更耗时耗力，虽然使用Schedulers.io()的地方，都可以使用Schedulers.newThread()，但是，Schedulers.newThread()的效率没有Schedulers.io()高。
Schedulers.computation()：用于CPU 密集型计算任务，即不会被 I/O 等操作限制性能的耗时操作，
Schedulers.trampoline()：在当前线程立即执行任务，如果当前线程有任务在执行，则会将其暂停，等插入进来的任务执行完之后，再将未完成的任务接着执行。
Schedulers.single()：拥有一个线程单例，所有的任务都在这一个线程中执行，当此线程中有任务执行时，其他任务将会按照先进先出的顺序依次执行。
AndroidSchedulers.mainThread()：在Android UI线程中执行任务，为Android开发定制。
Scheduler.from(@NonNull Executor executor)：指定一个线程调度器，由此调度器来控制任务的执行策略。
```
以上列出的是rxjava2中所有支持的Scheuler种类，Rxjava1中Schedulers.immediate()被Schedulers.trampoline()替换。

```
 Observable.create(new ObservableOnSubscribe<Integer>() {
           @Override
           public void subscribe(ObservableEmitter<Integer> e) throws Exception {
                for(int i = 0; i<5; i++){
                    System.out.println("发射线程:"+Thread.currentThread().getName()+"--->发射"+i);
                    Thread.sleep(1000);
                    e.onNext(i);
                }
                e.onComplete();
           }
          }).subscribeOn(Schedulers.io())
               .map(new Function<Integer, Integer>() {
                   @Override
                   public Integer apply(Integer integer) throws Exception {
                       System.out.println("处理线程:"+Thread.currentThread().getName()+"--->处理"+integer);
                       return integer;
                   }
               })
               .subscribeOn(Schedulers.newThread())
               .observeOn(AndroidSchedulers.mainThread())
               .subscribe(new Consumer<Integer>() {
                   @Override
                   public void accept(Integer integer) throws Exception {
                       System.out.println("接收线程:"+Thread.currentThread().getName()+"--->接收"+integer);
                   }
               });
               
//运行结果如下
System.out: 发射线程:RxCachedThreadScheduler-1---->发射:0
System.out: 处理线程:RxCachedThreadScheduler-1---->处理:0
System.out: 发射线程:RxCachedThreadScheduler-1---->发射:1
System.out: 接收线程:main---->接收:0
System.out: 处理线程:RxCachedThreadScheduler-1---->处理:1
System.out: 接收线程:main---->接收:1
```

### 背压
从上可知，数据流的发射、处理以及响应可能在各自线程中独立进行，上游的发射数据的时候，不知道下游是否处理完，所以会产生一种情况，发送事件的速度大于消费事件的速度，这样就会产生很多待处理的数据，不会被垃圾回收机制回收，而是存放在一个异步的缓存池中，如果缓存池中的数据一直得不到处理，越积越多，最后就会造成内存溢出，这便是背压问题。
```
Observable.create(new ObservableOnSubscribe<Integer>() {
           @Override
           public void subscribe(ObservableEmitter<Integer> e) throws Exception {
               int i = 0;
               while(true){
                   i++;
                   e.onNext(i);
               }
           }
       }).subscribeOn(Schedulers.newThread())
               .observeOn(Schedulers.newThread())
               .subscribe(new Consumer<Integer>() {
           @Override
           public void accept(Integer integer) throws Exception {
              Thread.sleep(3000);
               System.out.println(integer);
           }
       });
```
创建一个可观察对象Obervable在Schedulers.newThread()()的线程中不断发送数据，而观察者Observer在Schedulers.newThread()的另一个线程中每隔3秒接收一条数据，运行后，查看内存使用如下图：

![](/img/pl/rxjava2_backpressure.gif)

由于上下游分别在各自的线程中独立处理数据（如果上下游在同一线程中，下游对数据的处理会堵塞上游数据的发送，上游发送一条数据后会等下游处理完之后再发送下一条），而上游发送数据速度远大于下游接收数据的速度，造成上下游流速不均，导致数据累计，最后引起内存溢出。

#### Flowable
Flowable是为了解决背压（backpressure）问题，而在Observable的基础上优化后的产物，与Observable不是同一组观察者模式下的成员，Flowable是Publisher与Subscriber这一组观察者模式中Publisher的典型实现，Observable是ObservableSource/Observer这一组观察者模式中ObservableSource的典型实现。

既然Flowable是在Observable的基础上优化后的产物，Observable能解决的问题Flowable都能进行解决，何不抛弃Observable而只用Flowable呢。其实，这是万万不可的，他们各有自己的优势和不足。由于基于Flowable发射的数据流，以及对数据加工处理的各操作符都添加了背压支持，附加了额外的逻辑，其运行效率要比Observable低得多。

因为只有上下游运行在各自的线程中，且上游发射数据速度大于下游接收处理数据的速度时，才会产生背压问题。
所以，如果能够确定上下游在同一个线程中工作，或者上下游工作在不同的线程中，而下游处理数据的速度高于上游发射数据的速度，则不会产生背压问题，就没有必要使用Flowable，以免影响性能。


#### 背压策略
Flowable的异步缓存池不同于Observable，Observable的异步缓存池没有大小限制，可以无限制向里添加数据，直至OOM,而Flowable的异步缓存池有个固定容量，其大小为128。
BackpressureStrategy的作用便是用来设置Flowable通过异步缓存池存储数据的策略。

```
ERROR:在此策略下，如果放入Flowable的异步缓存池中的数据超限了，则会抛出MissingBackpressureException异常。

DROP:在此策略下，如果Flowable的异步缓存池满了，会丢掉将要放入缓存池中的数据。


BUFFER:此策略下，Flowable的异步缓存池同Observable的一样，没有固定大小，可以无限制向里添加数据，不会抛出MissingBackpressureException异常，但会导致OOM。

MISSING:此策略表示，通过Create方法创建的Flowable没有指定背压策略，不会对通过OnNext发射的数据做缓存或丢弃处理，需要下游通过背压操作符（onBackpressureBuffer()/onBackpressureDrop()/onBackpressureLatest()）指定背压策略。

LATEST:与Drop策略一样，如果缓存池满了，会丢掉将要放入缓存池中的数据，不同的是，不管缓存池的状态如何，LATEST都会将最后一条数据强行放入缓存池中。


Flowable.create(new FlowableOnSubscribe<Integer>() {
            @Override
            public void subscribe(FlowableEmitter<Integer> e) throws Exception {
                for(int i = 0; i<500; i++){
                    e.onNext(i);
                }
                e.onComplete();
            }
            //具体背压策略替换这里即可
        }, BackpressureStrategy.BUFFER)
                .subscribeOn(Schedulers.newThread())
                .observeOn(Schedulers.newThread())
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {
                        System.out.println(integer);
                    }
                });
                
        //onBackpressureXXX背压操作符
        //Flowable除了通过create创建的时候指定背压策略，也可以在通过其它创建操作符just，fromArray等创建后通过背压操作符指定背压策略。
        //onBackpressureBuffer()对应BackpressureStrategy.BUFFER
        //onBackpressureDrop()对应BackpressureStrategy.DROP
        //onBackpressureLatest()对应BackpressureStrategy.LATEST
        //这个与上面Flowable.create()方式创建效果是一样的
        Flowable.range(0, 500)
                .onBackpressureBuffer()
                .subscribeOn(Schedulers.newThread())
                .observeOn(Schedulers.newThread())
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {
                        System.out.println(integer);
                    }
                });
        
```

所以结合FlowableEmitter与Subscription，通过设置处理请求量(subscription.request)以及动态获取待处理请求量(FlowableEmitter.requested())，对Flowable做出改进，让其不会产生背压问题，也不会引起异常或者数据丢失。代码如下：
```
 Flowable.create(new FlowableOnSubscribe<Integer>() {
            @Override
            public void subscribe(FlowableEmitter<Integer> e) throws Exception {
                int i = 0;
                while(true){
                    //e.requested()获取下游未处理的事件量
                    if(e.requested() == 0) continue;   //此处添加代码，让Flowable按需添加代码
                    System.out.println("发射--->"+i);
                    i++;
                    e.onNext(i);
                }
            }
        }, BackpressureStrategy.MISSING)
                .subscribeOn(Schedulers.newThread())
                .observeOn(Schedulers.newThread())
                .subscribe(new Subscriber<Integer>() {
                    Subscription subscription;
                    @Override
                    public void onSubscribe(Subscription s) {
                       //s.request设置可处理的事件量，默认为0不进行处理
                        s.request(1);   //设置初始请求数据为1
                        subscription = s;
                    }
                    @Override
                    public void onNext(Integer integer) {
                        try {
                            Thread.sleep(1000);
                            System.out.println("接收--->"+integer);
                            subscription.request(1);//每接收到一条请求就增加一条请求量
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }

                    @Override
                    public void onError(Throwable t) {
                    }

                    @Override
                    public void onComplete() {
                    }
                });
```


### Observable简化版之Single、Comoplable以及Maybe
在Rxjava2中，Observale和Flowable都是用来发射数据流的，但是，我们在实际应用中，很多时候，需要发射的数据并不是数据流的形式，而只是一条单一的数据，或者一条完成通知，或者一条错误通知。在这种情况下，我们再使用Observable或者Flowable就显得有点大材小用，于是，为了满足这种单一数据或通知的使用场景，便出现了Observable的简化版——Single、Completable、Maybe。
#### Single
只发射一条单一的数据，或者一条异常通知，不能发射完成通知，其中数据与通知只能发射一个。

#### Completable
只发射一条完成通知，或者一条异常通知，不能发射数据，其中完成通知与异常通知只能发射一个

#### Maybe
可发射一条单一的数据，以及发射一条完成通知，或者一条异常通知，其中完成通知和异常通知只能发射一个，发射数据只能在发射完成通知或者异常通知之前，否则发射数据无效。

三者的调用方式与Observable类似，只是create方法中传递的对象不同而已，由于篇幅有限，这边就不给出代码示例，具体可参考：https://www.jianshu.com/p/66a55abbadef

## 总结
以上就是关于Rxjava2学习总结，关于背压这块我介绍的可能不是很清楚，可参照以下博客。

背压介绍：https://www.jianshu.com/p/ff8167c1d191

示例代码：https://github.com/penglian/Rxjava2Demo/



