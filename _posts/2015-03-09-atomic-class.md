---
layout: post
title: Java多线程中的原子类
description: "在Java多线程编程中经常要对同一个数据进行读写,这时候如果没有加上锁的话很容造成数据的不一致,原子类在这种情况下可以避免这种数据不一致性(通过synchronized来加锁也可以实现) "
category: Java
tags: ["Java", "多线程"]
---

{% include JB/setup %}

#### 先来看没有加锁的情况:

这个例子中我们想在pt类中对count进行累加, 两个线程各4次,最后应该输出8, 然而这个例子输出的结果从4-8都会出现,这就是典型的没有同步的例子.

{% highlight java %}
public class Main  {

    public static void main(String[] args) throws Exception {

        ProcessingThread pt = new ProcessingThread();
        Thread t1 = new Thread(pt, "t1");
        t1.start();
        Thread t2 = new Thread(pt, "t2");
        t2.start();
        t1.join();
        t2.join();
        System.out.println("Processing count=" + pt.getCount());

    }
}
class ProcessingThread implements Runnable {
    private int count;


    @Override
    public void run() {
        for (int i = 1; i < 5; i++) {
            processSomething(i);
            count++;  //非原子操作
        }
    }


    public synchronized int getCount() {
        return this.count;
    }


    private void processSomething(int i) {
        // processing some job
        try {
            Thread.sleep(i * 100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}
{% endhighlight %}

#### 再来看看原子类:

这个例子中,我们使用的AtomicInteger的两个操作是原子操作,因此不会出现数据不一致的情况.

{% highlight java %}
import java.util.concurrent.atomic.AtomicInteger;
 
 
public class JavaAtomic {
 
    public static void main(String[] args) throws InterruptedException {
 
        ProcessingThread pt = new ProcessingThread();
        Thread t1 = new Thread(pt, "t1");
        t1.start();
        Thread t2 = new Thread(pt, "t2");
        t2.start();
        t1.join();
        t2.join();
        System.out.println("Processing count=" + pt.getCount());
    }
 
}
 
 
class ProcessingThread implements Runnable {
    private AtomicInteger count = new AtomicInteger();
 
 
    @Override
    public void run() {
        for (int i = 1; i < 5; i++) {
            processSomething(i);
            count.incrementAndGet();  //原子操作
        }
    }
 
 
    public int getCount() {
        return this.count.get(); // 原子操作
    }
 
 
    private void processSomething(int i) {
        // processing some job
        try {
            Thread.sleep(i * 100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
 
}
{% endhighlight %}

#### synchronized例子

和上例结果一样,我们用synchronized关键字来实现

{% highlight java %} 
public class JavaAtomic {
 
    public static void main(String[] args) throws InterruptedException {
 
        ProcessingThread pt = new ProcessingThread();
        Thread t1 = new Thread(pt, "t1");
        t1.start();
        Thread t2 = new Thread(pt, "t2");
        t2.start();
        t1.join();
        t2.join();
        System.out.println("Processing count=" + pt.getCount());
    }
 
}
 
 
class ProcessingThread implements Runnable {
    private int count;
 
 
    @Override
    public void run() {
        for (int i = 1; i < 5; i++) {
            processSomething(i);
            synchronized(this) {
                count++;
            }
        }
    }
 
 
    public int getCount() {
        return this.count; // 原子操作
    }
 
 
    private void processSomething(int i) {
        // processing some job
        try {
            Thread.sleep(i * 100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
 
}
{% endhighlight %}
