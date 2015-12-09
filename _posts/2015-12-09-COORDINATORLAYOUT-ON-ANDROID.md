---
layout: post
title: NESTED SCROLLING WITH COORDINATORLAYOUT ON ANDROID
description: "NESTED SCROLLING WITH COORDINATORLAYOUT ON ANDROID"
category: Android
tags: ["Android"]
---

{% include JB/setup %}
 
> 原文：[https://lab.getbase.com/nested-scrolling-with-coordinatorlayout-on-android/](https://lab.getbase.com/nested-scrolling-with-coordinatorlayout-on-android/)

上一篇 [文章中](https://lab.getbase.com/introduction-to-coordinator-layout-on-android/) 我解释了什么是`CoordinatorLayout`以及如何自定义一个Behavior。有几个读者评论问如何写自定义的可滚动的View来和`CoordinatorLayout`以及`AppBarLayout`很好地进行交互。 有些读者还对为什么`AppBarLayout`只和`RecyclerView`交互，换成`ListView`却没有用。

我们来看看滑动`RecyclerView`时自动显示／隐藏`AppbarLayout`的效果吧：

![https://lab.getbase.com/wp-content/uploads/2015/10/demo3-1.gif](https://lab.getbase.com/wp-content/uploads/2015/10/demo3-1.gif)

[源代码](https://github.com/chrisbanes/cheesesquare)

查看一下`AppBarLayout`代码,我们会发现默认的Behavior的注解` @DefaultBehavior(AppBarLayout.Behavior.class)`

`AppBarLayout.Behavior`实现了以下几个方法，来对滑动事件作出响应：

{% highlight java %}

void    onNestedPreScroll(CoordinatorLayout coordinatorLayout, V child, View target, int dx, int dy, int[] consumed)
void    onNestedScroll(CoordinatorLayout coordinatorLayout, V child, View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed)
void    onNestedScrollAccepted(CoordinatorLayout coordinatorLayout, V child, View directTargetChild, View target, int nestedScrollAxes)
boolean onStartNestedScroll(CoordinatorLayout coordinatorLayout, V child, View directTargetChild, View target, int nestedScrollAxes)
void    onStopNestedScroll(CoordinatorLayout coordinatorLayout, V child, View target)

{% endhighlight %}


这就解释了为什么`AppBarLayout`放在一个`CoordinatorLayout`里面时会发生位置变动。 现在的问题是为什么`CoordinatorLayout`能够再它的子视图的滑动事件发生时得到通知。 我们来仔细看看`RecyclerView`类，也许能找到答案。

### SAY HELLO TO NESTEDSCROLLINGCHILD AND NESTEDSCROLLINGPARENT INTERFACES.

我们看到`RecyclerView`实现了`NestedScrollingChild`接口，Doc是这么写的：

>  This interface should be implemented by View subclasses that wish to support dispatching nested scrolling operations to a cooperating parent ViewGroup.

大概意思就是说如果View的子类要实现分发嵌套（Nested）的事件给父容器，就实现这个接口。

要接收嵌套（Nested）的滑动操作，`CoordinatorLayout`需要实现`NestedScrollingParent`接口。

> This interface should be implemented by ViewGroup subclasses that wish to support scrolling operations delegated by a nested child view.


我们再深入进入`NestedScrollingChild`和`NestedScrollingParent`的通信过程。

当子View将要开始滑动时（会对应到onTouch的down事件），它就会执行onScrollStarted方法，在滑动的每一步都会执行两个方法`dispatchPreScroll`和`dispatchScroll`， 第一个方法为父容器在子View消费滑动操作之前去消费其中的一部分或者全部。如果父容器返回true，那么子View应该减少它的滑动量，这个量是由父容器消费的。`dispatchNestedScroll`方法报告滑动进度给父容器，包括已消费的和未消费的滑动量。父容器可以根据情况处理：

> An implementation may choose to use the consumed portion to match or chase scroll position of multiple child elements, for example. The unconsumed portion may be used to allow continuous dragging of multiple scrolling or draggable elements, such as scrolling a list within a vertical drawer where the drawer begins dragging once the edge of inner scrolling content is reached.


`CoordinatorLayout`就扮演了一个代理。它接收到从子View来的回调，并转发给它的Behavior类。

当滑动结束时，子View会调用`onScrollStoped`。

### IMPLEMENTING CUSTOM NESTEDSCROLLINGCHILD VIEW.

我们既然知道它是怎么工作的了，我们就可以实现一个示例程序，其中包含一个自定义View，它会检测滑动，并转发给它的`CoordinatorLayout`。

Let’s start with the View
public class NestedScrollingChildView extends View implements NestedScrollingChild, OnGestureListener

开始代码：

{% highlight java %}

public class NestedScrollingChildView extends View implements NestedScrollingChild, OnGestureListener

{% endhighlight %}

Android团队里的好人们想让这个工作稍微简单一点，所以它们做了一个`NestedScrollingChildHelper`.

 > Helper class for implementing nested scrolling child views compatible with Android platform versions earlier than Android 5.0 Lollipop (API 21).

我们需要做的就是创建一个helper的示例，并实现适当的方法。
默认的nested滑动是关闭的，我们可以在构造函数里启用它。


{% highlight java %}

setNestedScrollingEnabled(true);

{% endhighlight %}

现在我们要检测滑动，我们用`GestureDetectorCompat`来减少代码量

{% highlight java %}

@Override
public boolean onTouchEvent(MotionEvent event){
  return mDetector.onTouchEvent(event);
}

{% endhighlight %}

在`onDown`方法里，我们要调用`onStartScrolling`，然后传递垂直轴的flag作为参数。

{% highlight java %}

@Override
public boolean onDown(MotionEvent e) {
  startNestedScroll(ViewCompat.SCROLL_AXIS_VERTICAL);
  return true;
}

{% endhighlight %}

从`onScroll`方法里，我们可以调用`dispatchNestedPreScroll`，传递计算好的distanceY，然后调用`dispatchNestedScroll`。由于我们的View其实并不会滑动，因此，我们0作为我们消费和未消费的滑动值。

{% highlight java %}

@Override
public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) {
  dispatchNestedPreScroll(0, (int) distanceY, null, null);
  dispatchNestedScroll(0, 0, 0, 0, null);
  return true;
}

{% endhighlight %}


最后一部分是当滑动过程结束时，调用`stopNestedScroll`。由于`GestureDetectorCompat`没有提供合适的回调，我们要在`onTouchEvent`函数中自定义一个实现。

{% highlight java %}


@Override
public boolean onTouchEvent(MotionEvent event){
  final boolean handled = mDetector.onTouchEvent(event);
  if (!handled && event.getAction() == MotionEvent.ACTION_UP) {
    stopNestedScroll();
  }
  return true;
}

{% endhighlight %}


为了保持简单，本例中不处理Fling操作（快速滑动后的惯性）。

And voilà

![https://lab.getbase.com/wp-content/uploads/2015/10/demo.gif](https://lab.getbase.com/wp-content/uploads/2015/10/demo.gif)


完整的源码在这里：[https://github.com/ggajews/nestedscrollingchildviewdemo](https://github.com/ggajews/nestedscrollingchildviewdemo) 如果你还希望让`ListView`也和`CoordinatorLayout`配合，你要自定义 `ListView`实现`NestedScrollingChild`接口。








