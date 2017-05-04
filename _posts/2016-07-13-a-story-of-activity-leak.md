---
layout: post
title: 记一次Android中Activity内存泄露的调查
description: "本文通过一次内存泄露的分析和调查过程，来讲述一下一个比较通用的内存泄露分析过程和查找方式"
category: Android
tags: ["Android","内存泄露","Java"]
---

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

### 0. 起因

最近刚刚接手新的直播项目，在看代码的过程中观察了一下Android Studio中的内存状况，发现使用的时间越长，内存占用越多，第一反应就是可能发生内存泄露。直播项目一般都有列表页和直播页，每次打开直播页，内存占用会增大，而退出直播页后手动GC，发现内存占用并没有变小，所以初步判断有可能直播播放页发生了内存泄露。

### 1. Android Studio中内存快照
Android Studio中自带了一个叫Android Monitor的工具，这个工具包含Memory、CPU、Network和GPU的监控，其中Memory监控的工具上还提供了内存快照以及内存分配跟踪工具，这是我们查看内存情况的重要工具，通常从内存状态曲线可以判断一些明显的内存泄露问题。我们这里将会使用内存快照功能来检查内存中的对象，并确认内存泄露是否发生。
功能按键如下图：
![pic](https://github.com/junyuecao/private-static/blob/master/20160713a.png?raw=true)

#### 1.1 使用内存快照

这里我们要判断是否泄露的类名叫LiveWatchActivity，反复打开关闭LiveWatchActivity，然后进行一次手动GC，此时抓取一次内存快照后，会打开一个如下图的界面：
![pic](https://github.com/junyuecao/private-static/blob/master/20160713b.png?raw=true)

其中上半部分左侧窗口为堆中包含的类列表，上半部分右侧为左侧的类对应的所有实例，下方为引用树，最右侧为分析任务窗口

#### 1.2 如何确认内存泄露
对于Activity来说，可以使用右侧窗口中的分析功能，它会自动分析出已经泄露的Activity，非常方便。如上图我们发现了6个LiveWatchActivity和一个SplashActivity泄露。

如果不是Activity，我们需要在左侧窗口中查看某个类对应的对像的个数来判断，如果某个类对应的对像在多次快照的次数越来越多，根据代码逻辑如果判断为不正常的话，就可以认为这个类的对象发生了泄露。

下方的![pic](https://github.com/junyuecao/private-static/blob/master/20160713c.png?raw=true) 图标表示上方的对象由这个对象支配，右键点击这个对象，然后选择Go to instance可以跳转到支配它的对象，重复选择Go to instance，你将看到防止GC的GCRoot，它的图标是：![pic](https://github.com/junyuecao/private-static/blob/master/20160713d.png?raw=true) ，这个就是导致内存无法回收的真凶了。如果没有看到它的支配对象，那就要请出我们的另一个工具Eclipse Memory Analyzer了。

### 2. 使用Eclipse Memory Analyzer分析内存
首先下载它：[www.eclipse.org/mat/downloads.php](www.eclipse.org/mat/downloads.php)
然后安装它：这里不啰嗦了

以下用mat代替Eclipse Memory Analyzer

#### 2.1 转换格式
Android Studio的内存快照是无法直接用mat打开的，我们需要转换成mat使用的格式，具体操作方式为：Android Studio最左侧的Project旁边有个Capture，点开后选择我们需要的快照，右键点击选择`Export to standard .hprof`，然后选择路径保存。
打开mat，从File->Open Heap Dump，即可打开这个快照。

#### 2.2 查找GC Root

打开快照后OverView界面下方有个Actions 区域，选择Dominator Tree，打开一个新的Dominator Tree窗口，最上方输入要过滤的类，这里我输入`LiveWatchActivity`，就列出了我们刚才找到的泄露的对象了：
![pic](https://github.com/junyuecao/private-static/blob/master/20160713e.png?raw=true) 

任意选择一个，选择下图的菜单项，就可以看到GC Root对象了，这里我们过滤了WeakReference的对象。
![pic](https://github.com/junyuecao/private-static/blob/master/20160713f.png?raw=true) 

### 3. 找到泄露原因

在GC Root结果中，带一个小橙色点的就是最终的GC Root：
![pic](https://github.com/junyuecao/private-static/blob/master/20160713g.png?raw=true) 

从上图中我们看出实际上这个Activity在主线程的MessageQueue中被引用了。很有可能是handler导致的内存泄露。

于是我们再回到Android Studio，找到主线程的MessageQueue（mat也可以查看对象的内容，但是感觉没有AS的好用）。

![pic](https://github.com/junyuecao/private-static/blob/master/20160713h.png?raw=true) 

这里很关键的内容就是callback:`LiveGiftHolder$2`，找到这个匿名内部类，就基本知道是什么原因了。在AS中打开这个`LiveGiftHolder`， 按CMD+F12（windows的可以自己查一下）打开FileStructure：就可以看到这个匿名内部类是哪个了：

![pic](https://github.com/junyuecao/private-static/blob/master/20160713i.png?raw=true) 

这里的`LiveWatchManager`持有了`LiveWatchActivity`，于是最终我们找到了导致内存泄露的地方，其实是一个递归`Handler.postDelayed()`导致的无法释放Activity的问题。

### 4. LeakCanary不是万能的

LeakCanary其实就是像我们一样通过Dump来寻找GC Root，但是在本例中我发现有些泄露的对象没有Dominator，这种情况下LeakCanary也无法发现泄露问题。不要以为用了LeakCanary就没有内存泄露，还是要处处小心谨慎。



