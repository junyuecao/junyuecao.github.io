---
layout: post
title: 摆脱不同分辨率的束缚 - 使用Android Studio的Vector Asset做安卓图标
description: 以前做Android App时,我们需要对不同的分辨率做适配的话, 每个图标都需要做5个分辨率(ldpi设备很少了,不然就得6个), 非常麻烦. Android Studio从1.4开始支持使用适量图(Vector Asset),从此图标只需要做一套矢量图就可以了, Android Studio会自动生成不同分辨率的图标.开发者只需要像普通图标一样使用即可.
category: Android
tags: ["Android"]
---

{% include JB/setup %}

### 史前做法

以前做Android App时,我们需要对不同的分辨率做适配的话, 每个图标都需要做5个分辨率(ldpi设备很少了,不然就得6个), 非常麻烦. 在工具方面而且生成不同分辨率的图标的工具并没有一个做的很好的,导致我们在开发过程中,在处理图片上耗费了大量的时间和精力.

### 一切变得更加容易

Android Studio从1.4版本(需要Gradle插件1.5以上),加入了Vector Asset功能,这个功能可以让你导入矢量图,并指定它的dp大小,Android Studio和Gradle将自动生成不同分辨率对应的图片,并打包至APK.

在我们的源码里面只有一个xml文件:

![Vector Asset文件](https://raw.githubusercontent.com/junyuecao/private-static/master/vector-asset.png)

在生成的apk文件里,对应的所有DPI都会有一个png图片:

![Vector Asset文件](https://raw.githubusercontent.com/junyuecao/private-static/master/20160106a.png)

### 如何使用矢量图标(Vector Asset)?

Android Studio提供了一个向导来生成矢量图标.

在drawable目录右键点击后,点击New,点击Vector Asset,如下图, 或者使用File菜单(不再赘述):

![Vector Asset文件](https://raw.githubusercontent.com/junyuecao/private-static/master/20160106b.png)

在弹出的窗口中有两个选择:

1.选择Android Studio自带的图标库里的图标
2.选择本地svg文件

选择完成后可以设置大小, 透明度, 是否支持RTL布局等.
 
注意,图标颜色在这里是无法修改的,需要生成之后到svg文件里修改图标的颜色.或者使用代码修改

{% highlight java %}

    Drawable drawable = getDrawable;
    drawable = DrawableCompat.wrap(drawable);
    DrawableCompat.setTint(drawable, Color.GREEN);
    
    DrawableCompat.setTintMode(drawable, PorterDuff.Mode.SRC_IN);
    
    // 如果要恢复原始颜色的话
    DrawableCompat.unwrap(Drawable drawable);

{% endhighlight %}

![Vector Asset文件](https://raw.githubusercontent.com/junyuecao/private-static/master/20160106c.png)

点击下一步,可以选择这个图标所在的目录.

然后就可以在代码中使用这个图标了,怎么样是不是很简单呢!






