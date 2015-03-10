---
layout: post
title: 如何理解MeasureSpec
description: ""
category: Android
tags: ["Java", "Android"]
---

{% include JB/setup %}

在自定义View的时候,我们经常会用到onMeasure这个函数,这个函数:

{% highlight java %}
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)
{% endhighlight%}

这个函数接收的两个参数表示的父View希望子View如何计算自己的大小..很多人对MeasureSpec.EXACTLY, MeasureSpec.AT_MOST, MeasureSpec.UNSPECIFIED这三个类型有疑问.如何来理解呢?

我们看下面的代码(来自ViewGroup.java):

{% highlight java %}
/**
 * Does the hard part of measureChildren: figuring out the MeasureSpec to
 * pass to a particular child. This method figures out the right MeasureSpec
 * for one dimension (height or width) of one child view.
 *
 * The goal is to combine information from our MeasureSpec with the
 * LayoutParams of the child to get the best possible results. For example,
 * if the this view knows its size (because its MeasureSpec has a mode of
 * EXACTLY), and the child has indicated in its LayoutParams that it wants
 * to be the same size as the parent, the parent should ask the child to
 * layout given an exact size.
 *
 * @param spec The requirements for this view
 * @param padding The padding of this view for the current dimension and
 *        margins, if applicable
 * @param childDimension How big the child wants to be in the current
 *        dimension
 * @return a MeasureSpec integer for the child
 */
public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
    int specMode = MeasureSpec.getMode(spec); // 一般来说这是父View传进来的
    int specSize = MeasureSpec.getSize(spec); // 同上

    int size = Math.max(0, specSize - padding); //去掉padding得到内容的大小

    int resultSize = 0;
    int resultMode = 0;

    switch (specMode) {
    // Parent has imposed an exact size on us
     // childDimension是layout文件里设置的大小:可以是MATCH_PARENT, WRAP_CONTENT或者具体大小, 代码中分别对三种做不同的处理
    case MeasureSpec.EXACTLY:
        if (childDimension >= 0) {
            //如果childDimension是具体大小,那么就按照这个大小来, Mode为Exactly,这样子View就知道要按照填写的大小来draw
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY; 
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size. So be it.
            // 子View填的是Match_parent, 那么父View就给子view自己的size(去掉padding), 然后告诉子view这是Exactly的大小
            resultSize = size; 
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size. It can't be
            // bigger than us.
            // 子View 填的是wrap_Content,那么父View就告诉子View自己的size(去掉padding), 然后告诉子View, 你最多有这么多, 你自己决定大小
            resultSize = size; 
            resultMode = MeasureSpec.AT_MOST;
        }
        break;

    // Parent has imposed a maximum size on us
    case MeasureSpec.AT_MOST:
        if (childDimension >= 0) {
            // Child wants a specific size... so be it
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size, but our size is not fixed.
            // Constrain child to not be bigger than us.
            // 如果子View是Match_parent,但是父View是告诉子View, 你最多有size这么多, 那么就告诉子View,你最多有父View的size
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size. It can't be
            // bigger than us.
            // 基本和上一个情况一样
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        }
        break;

    // Parent asked to see how big we want to be
    case MeasureSpec.UNSPECIFIED:
        if (childDimension >= 0) {
            // Child wants a specific size... let him have it
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size... find out how big it should
            // be
            resultSize = 0;
            resultMode = MeasureSpec.UNSPECIFIED;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size.... find out how
            // big it should be
            resultSize = 0;
            resultMode = MeasureSpec.UNSPECIFIED;
        }
        break;
    }
    return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
}
{% endhighlight%}

上面的代码会在Measure子View的时候调用, 这段代码大致说明了MATCH_PARENT和WRAP_CONTENT与EXACTLY和AT_MOST的关系.

***仔细阅读上面的代码,你一定会对onMeasure这个函数有更深的理解***

