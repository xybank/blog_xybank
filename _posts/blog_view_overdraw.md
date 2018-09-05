---
title: View的过度绘制简述
date: 2018-09-03 12:00:00
tags: [过度绘制]
categories: 彭连
description: 简要介绍Android中View的过度绘制以及常用的检测方法与防范
---

## 简述
话不多说，先来两张图：

图一：
![](/img/pl/view_overdraw_demo1.jpg)

图二：

![](/img/pl/view_overdraw_demo2.jpg)

以上两张是通过Android手机的*调试GPU过度绘制*功能显示的两个不同的应用检测界面，颜色越红说明过度绘制越严重。第一张以绿色与蓝色居多，还算OK；而第二张则是粉红色甚至红色为主，说明过度绘制比较严重。

### 什么是过度绘制
过度绘制描述的是屏幕上的某个像素点在同一帧的时间内被绘制了多次。在多层的UI结构里面，如果不可见的UI也在做绘制操作，会导致某些像素区域被绘制了多次，同时也会浪费大量的CPU与GPU资源，绘制示意图如下展示：

![](/img/pl/view_overdraw_invilate.png)

这里可以看出是一层层绘制，搞清楚过度绘制之前，还是先了解Android的渲染机制吧。

### Android渲染机制
#### 1、Android如何将代码渲染到手机屏幕上
我们知道手机是由许多像素点构成的通过显示每个像素点不同的颜色，然后组成各种图像；Activity的界面之所以可以被绘制到屏幕上其中有一个很重要的过程就是栅格化，栅格化简单来说就是将向量图转化为机器可以识别的位图的过程，其中很复杂也很耗时。

GPU就是用来加速栅格化的过程的，下面用一张图来介绍下大致的渲染过程：

![](/img/pl/view_overdraw_resterization.jpg)

从上面的图可以看出，CPU会先把UI组件计算成polygons(多边形)和texture(纹理)，然后再交给GPU栅格化渲染，最后GPU再将数据传递给屏幕，有屏幕进行绘制显示。当然，从CPU到GPU还需要进行Opengl ES的处理，这也是很复杂的一个过程。

因此我们可以看出android的渲染依赖两个核心组件：
* CPU：负责Messure、Layout、Record以及Execute等计算操作（容易产生不必要的重绘）
* GPU：负责Rasterization（栅格化）操作（容易产生过度绘制问题）

在GPU方面，最常见的问题就是我们所说的过度绘制，通常是在像素着色过程中，通过其它工具进行后期着色时浪费了GPU的处理时间。

#### 2、手机像素点数据哪里来？Frame Buffer和Back Buffer
手机屏幕是由许多的像素点组成的，通过让每一个屏幕显示不同的颜色，可以组合成各种各样的图像。这些像素点的颜色是从哪里来的？

GPU控制的一块缓冲区，这块缓冲区叫做Frame Buffer(也就是帧缓冲区)。你可以把它理解成一个二维数组，数组中每一个元素对应着手机屏幕的一个像素点，元素的值代表着屏幕上对应的像素点要显示的颜色。优化屏幕画面不断变化，需要这个Buffer不断的更新数据，一个FrameBuffer肯定是应接不暇的，因此GPU除了Frame Buffer，用以交给手机屏幕进行绘制外，还有一个缓冲区，叫Back Buffer，这个Back Buffer用以交给你得应用，让你往里面填充数据。GPU会定期交换Back Buffer与Frame Buffer，也就是让Back Buffer变成Frame Buffer交给屏幕进行绘制，让Frame Buffer变成Back Buffer交给你得应用进行绘制。交换的频率是60次/s，这就与屏幕硬件电路的刷新频率保持了同步。如下图所示：

![](/img/pl/view_overdraw_buffer.png)

关于缓冲机制如双缓冲、三缓冲这里就不进行介绍了，有兴趣的可以自行查阅资料。

#### 3、关于VSync
刷新屏幕的时机就由VSync信号来控制，由于人眼与大脑的协作无法感知超过60fps的画面更新，因此需要在16ms内完成屏幕刷新的所有逻辑，否则就会出现画面丢失造成卡顿。

VSync(Vertical Synchronization)，就是所谓的垂直同步。在Android也沿用了这个概念，我们也可以把它理解为"帧同步"。这个用保证CPU、GPU生成帧的速度与display刷新的速度保持一致。Android系统每16ms就会发出VSync信号触发UI渲染更新。上面提到屏幕一秒刷新60次，这就要求CPU与GPU每秒要有处理60帧的能力，一帧花费的时间在16ms内。那么在Android系统中是如何利用VSYNC进行工作的呢？如下图

![](/img/pl/view_overdraw_vsync.jpg)

设计预期是这样的一个过程：在VSYNC信号刚发出时，Android系统就立刻开始了下一帧数据的处理了，这样就不会浪费时间了。图中先显示第0帧，在这16ms内，CPU与GPU已经开始准备下一帧的数据了，赶在下一个VSYNC信号到来时，GPU渲染完成，及时交换数据，display绘制显示完成，每一帧如果可以井然有序的进行，那么用户就不会感到卡顿了，看起来很美妙，难道不是吗？

## 过度绘制的检测与防范
通过上面的一些理论学习，我们更好了解了关于过度绘制的一些细节，下面将来介绍以下如何解决过度绘制的常见工具和套路。

### 常见工具
#### 手机检测工具
按照以下步骤打开Show GPU Overdraw的选项：设置——>开发者选项——>调试GPU过度绘制——>显示GPU过度绘制。

每个颜色的说明如下：

```
原色：没有过度绘制
蓝色：1 次过度绘制
绿色：2 次过度绘制
粉色：3 次过度绘制
红色：4 次及以上过度绘制
```

过度绘制的存在会导致界面显示时浪费不必要的资源去渲染看不见的背景，或者对某些像素区域进行多次绘制，就会导致界面加载或者滑动时的不流畅、掉帧，对于用户来说就是App特别的卡顿。为了提升用户体验，提升应用的流畅性，优化过度绘制的工作还是很有必要的。

#### Android Studio检测工具
打开步骤：Tools-> Android -> Android Device Monitor，如下图：

![](/img/pl/view_overdraw_monitor.png)


#### HierarchyViewer
进行UI布局复杂程度及冗余等分析，要使用这个工具需要在虚拟机上才能work，不然会报

```
Unbale to get view server version from device
```

如果需要真机使用这个功能Android SDK开发团队提供了ViewServer这个开源库，项目地址https://github.com/romainguy/ViewServer，需要在Activity的onCreate、onDestroy以及onResume这三个方法中调用ViewServer的对应的方法即可在真机环境下使用HierarchyViewer工具。

#### 布局分析器

![](/img/pl/view_overdraw_layoutparse1.png)

这个工具通常也可以分析别人app UI的实现，非常强大

![](/img/pl/view_overdraw_layoutparse2.png)

你可以查看当前UI截图的任意控件，比如觉得图片那块过度绘制这么严重，点击看一下，可以看到对应的布局、资源ID，全局查找一下，就可以到对应的位置去看看代码，这样就可以通过可视化快速定位到问题代码，进行修复。

### 防范套路
通过以上的工具找到问题后就是如何去处理了，下面总结了一些常用的套路。

#### 套路一、去掉window的默认背景
当我们使用Android自带的一些主题时，window会被默认添加一个纯色的背景，这个背景是被DecorView持有的。当我们的自定义布局又添加了一张背景图或者设置了背景色，那么DecorView的background对我们来说是无用的，但是它会产生一次overdraw，带来绘制性能消耗。去掉window背景可以在onCreate（）的setContentView（）之前调用。

```
getWindow().setBackgroundDrawable(null);
```

或者在theme中添加

```
android:windowground = "null"
```
#### 套路二、去掉其它不必要的背景
有时候为了方便会给Layout设置一个整体的背景，再给子View设置背景，这里也会造成重叠，如果子View宽度为match_parent，可以看到完全覆盖了Layout的一部分，这里可以通过分别绘制背景来减少重绘。再比如如果设置的是seletor的背景，将normal状态的color设置为“@android:color/transparent”，也同样可以解决问题。这里只简单的举了两个例子，我们在开发过程中的一些习惯性的思维定式会带来不经意的过度绘制，所以开发过程中我们为某个View或者ViewGroup设置背景时，先考虑下是否真的有必要，或者想下这个背景能否分段设置在子View上，而不是图方便直接设置在根View上。

#### 套路三、clipRect的使用
我们可以通过canvas.clipRect()来帮助系统识别那些可见的区域。这个方法可以指定一个矩形区域，只有在一个区域内才会被绘制，其它区域会被忽略。这个api可以很好的帮助那些有多组重叠组件的自定义View来控制显示的区域。同时clipRect方法还可以帮助节约CPU与GPU的资源，在clipRect区域之外的绘制指令都不会被执行，那些部分区域在矩形区域内的组件，仍然会得到绘制。

#### 套路四、ViewStup
ViewStup称之为延迟化加载，在多数情况下，程序无需显示的指示ViewStup所指向的布局文件，只有在特定的，某些较少的条件下，此时ViewStup所指向的布局文件才需要被inflate，且此布局文件直接在当前ViewStup替换掉，具体是通过ViewStup.inflate()或者ViewStup.setVisibility(View.VISIBLE)来设置的；常见的比如网络加载布局。

#### 套路五、Merge标签
Merge标签可以干掉一个层级。Merge的作用很明显，但是也有一些使用条件的限制。有两种条件我们可以使用Merge标签来做容器控件。第一种子视图不需要指定任何针对父布局的属性，就是说父容器仅仅就是个容器，子视图只需添加到父视图上用于显示就行了。另外一种是加入需要在LinearLayout中嵌入一个布局，而恰恰这个布局的根节点也是LinearLayout，这样就多了一层没用的嵌套，无疑这样就会拖慢程序速度。而这个时候我们使用Merge标签就可以避免那样的问题。另外merge只能作为xml的根标签使用，当inflate以开头的布局文件时，必须指定一个父ViewGroup，并且设置attachRoot为true。不常用的UI被设置成GONE，比如异常的错误页面，如果有这类问题，我们需要使用标签，代替GONE提高UI性能。毕竟View的VISIBLE/GONE是会引起布局重绘的。

#### 套路六、include标签
include标签能够重用布局。

还可以尝试使用RelativeLayout以及ConstraintLayout来减少布局的嵌套，提高UI性能。

## 总结
事情虽小，也不是什么大的难点，但是这个背后的牵涉的面还是很广的，还是值得去总结总结，有助于以点到面的构建自己的知识体系。


## 参考链接
* http://hukai.me/android-performance-patterns/
* https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE
* https://developer.android.com/studio/profile/dev-options-overdraw.html
* http://www.jianshu.com/p/a769a6028e51
* http://blog.csdn.net/yanbober/article/details/48394201
* http://androidperformance.com/2015/01/13/android-performance-optimization-overdraw-2.html
* http://blog.csdn.net/xyz_lmn/article/details/14524567

