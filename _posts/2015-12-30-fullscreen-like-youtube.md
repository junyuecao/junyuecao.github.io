---
layout: post
title: Android 实现类似YouTube的全屏播放器
description: 实现Android上YouTube的全屏播放
category: Android
tags: ["Android"]
---



### 前言

体验了几款Android播放器，在全屏播放这一块，作为Google旗下的YouTube在播放体验上一向是做的非常优秀的，YouTube在全屏的时候视频区域是占据整个屏幕的，包括导航条和状态栏都不会影响视频区域的大小，这样做的好处就是能够实现完全的全屏，避免在播放的时候还得看着三大金刚的尴尬，点击视频出现视屏控件的时候导航条和状态栏也一起出现，同时视频大小依然不受影响，在大屏手机上视觉效果非常好，体验也是非常好的。Youku在早期的时候虽然播放的时候导航条是消失的，但是点击视频区域，导航条出现时，却压缩了视频的区域，导致视频区域闪一下，现在的优酷视频区域已经不再闪了，但是他的视频播放控件依然还是会有一个大小的变化，细心的人应该很容易察觉到。

在Kitkat 里加入了Translucent的导航条和状态栏，来实现全屏的App页面。我们正是利用这个特性来实现这种全屏的方式。

### TL;DR

简单来说就是：

1. Activity Translucent
2. 播放控件fitSystemWindows

### 实现

要实现这样的全屏播放，主要的两个关键字就是：translucent 和 fitSystemWindows

首先我们来看style(4.4以上才支持)

{% highlight xml %}
	
	<?xml version="1.0" encoding="utf-8"?>
	<resources>
	    <style name="AppThemeTranslucent" parent="AppTheme">
	        <item name="android:windowTranslucentStatus">true</item>
	        <item name="android:windowTranslucentNavigation">true</item>
	    </style>
	</resources>

{% endhighlight %}

要让这个视频全屏播放，我们必须让视频所在的Activity的style里的`android:windowTranslucentStatus` 和 `android:windowTranslucentNavigation`设置为true。设置后Activity的状态栏和导航栏都不会占据空间（也因为这个在非全屏播放时要计算好内容区域，防止内容被状态栏和导航栏遮挡）。

将我们的视频容器设置为全屏后视频容器就会占据整个屏幕， 同时我们需要在隐藏控件的时候隐藏导航栏和状态栏：

{% highlight java %}

	public void toFullScreenMode() {

        // The UI options currently enabled are represented by a bitfield.
        // getSystemUiVisibility() gives us that bitfield.
        int newUiOptions = 0;

        // Navigation bar hiding:  Backwards compatible to ICS.
        if (Build.VERSION.SDK_INT >= 14) {
            newUiOptions = View.SYSTEM_UI_FLAG_HIDE_NAVIGATION;
            newUiOptions = newUiOptions | View.SYSTEM_UI_FLAG_LOW_PROFILE;
        }

        // Status bar hiding: Backwards compatible to Jellybean
        if (Build.VERSION.SDK_INT >= 16) {
            newUiOptions = newUiOptions | View.SYSTEM_UI_FLAG_FULLSCREEN;
        }

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            newUiOptions = newUiOptions | View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY;
        }

        mContext.getWindow().getDecorView().setSystemUiVisibility(newUiOptions);
    }

{% endhighlight %}

这样，在全屏的时候就会看到一个覆盖整个屏幕的视频。

但是我们点击视频后出现控件，就会发现控件也是全屏的，那么就会有一部分被导航条和状态栏遮挡，怎么办呢？这个时候就是我们的fitSystemWindows出场的时候啦。

在我们的MediaController的`show`里增加一些代码：

{% highlight java %}

	if (mFullscreen) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            setFitsSystemWindows(true);
        }
    } else {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            setFitsSystemWindows(false);
            setPadding(0, 0, 0, 0);
        }
    }

{% endhighlight %}

关于setFitsSystemWindows可以简单地理解为当为true的时候这个View的内容区域会扣除系统UI区域，因此我们的视频控制区的绘制就会排除状态栏和导航栏。就像下面这样：
![全屏状态](https://raw.githubusercontent.com/junyuecao/private-static/master/screenshots1.png)

在全屏状态下时，对于我们的MediaController，我们需要设置为setFitsSystemWindows(true)，在非全屏状态下时，我们需要设置为setFitsSystemWindows(false)，并清除它的padding值（setFitsSystemWindows(true)会自动设置内容的padding）。

### 结论
就这样通过setFitsSystemWindows 和 Translucent特性，我们就实现了YouTube类似的全屏播放，彻底解决导航条在全屏下碍眼的问题。科科。





