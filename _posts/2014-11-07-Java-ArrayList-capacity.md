---
layout: post
title: ArrayList里面的空间自动扩充
description: "ArrayList里面的空间自动扩充"
category: Java
tags: ["Java"]
---



我们知道在使用ArrayList时，我们无需关心他的大小，只需要往里放置对象即可，那么它是如何做的空间自动扩充的呢？
首先，我们在构造函数中可以传入一个int值，这个值表示的就是初始的容量，如果没有传这个值，那么默认的容量是0。


{% highlight java %}

/**
 * The elements in this list, followed by nulls.
 */
transient Object[] array;
/**
 * Constructs a new instance of {@code ArrayList} with the specified
 * initial capacity.
 *
 * @param capacity
 *            the initial capacity of this {@code ArrayList}.
 */
public ArrayList(int capacity) {
    if (capacity < 0) {
        throw new IllegalArgumentException("capacity < 0: " + capacity);
    }
    array = (capacity == 0 ? EmptyArray.OBJECT : new Object[capacity]);
}

/**
 * Constructs a new {@code ArrayList} instance with zero initial capacity.
 */
public ArrayList() {
    array = EmptyArray.OBJECT;
}
{% endhighlight %}


通过上面的代码我们看到如果capacity是0的话，和不传int参数的作用是一样的，如果int值是大于0的那么,我们保存对象用的数组就是这个确切的值。

那么如果说我们往里存入对象时，空间不够用了，会发生什么呢？

{% highlight java %}
/**
 * Adds the specified object at the end of this {@code ArrayList}.
 *
 * @param object
 *            the object to add.
 * @return always true
 */
@Override public boolean add(E object) {
    Object[] a = array;
    int s = size;
    if (s == a.length) {
        Object[] newArray = new Object[s +
                (s < (MIN_CAPACITY_INCREMENT / 2) ?
                 MIN_CAPACITY_INCREMENT : s >> 1)];
        System.arraycopy(a, 0, newArray, 0, s);
        array = a = newArray;
    }
    a[s] = object;
    size = s + 1;
    modCount++;
    return true;
}
{% endhighlight %}


这里我们看到了如果说size 和数组的长度一样了，也就是说数组满了，那么会重新new出一个数组，这个数组大小是`s + (s < (MIN_CAPACITY_INCREMENT / 2) ?  MIN_CAPACITY_INCREMENT : s >> 1)`，而这个`MIN_CAPACITY_INCREMENT`常量的大小是12:


{% highlight java %}
/**
 * The minimum amount by which the capacity of an ArrayList will increase.
 * This tuning parameter controls a time-space tradeoff（时间-空间权衡的方法）. This value (12)
 * gives empirically good results（这个12是经验上得出来比较好的初始值） and is arguably consistent with the
 * RI's specified default initial capacity of 10: instead of 10, we start
 * with 0 (sans allocation) and jump to 12.
 */
private static final int MIN_CAPACITY_INCREMENT = 12;
{% endhighlight %}


因此如果是刚开始size小于6的话，那么这个新的数组长度就会变成12，如果size已经比大于货等于12了，那么这个新的数组就是原来的两倍大小。

在addAll方法中还有另一种逻辑，但是和这个逻辑基本是一致的，有兴趣的可以去看看java源码。