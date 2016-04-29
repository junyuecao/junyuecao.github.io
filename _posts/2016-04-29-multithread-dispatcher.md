---
layout: post
title: Volley学习之Java多线程分发器实例
description: "在Android的网络框架Volley中使用了多线程分发网络请求的方法, 这种方法可以看作是一个生产者消费者问题, 让我们也跟着来实现一个这样的小分发器"
category: Java
tags: ["Java", "Android", "Volley", "多线程"]
---


#### 前言

Volley 是个优秀的网络请求框架，它的设计决定了它适合多个并行的小规模的请求，比如大量的接口请求。今天我们把Volley身上所有的皮肉都剃掉，只保留一个骨架，这个骨架对于我们理解如何实现一个多线程分发器很有帮助，首先我们先来看看Volley的架构图：

![Volley](https://github.com/junyuecao/private-static/blob/master/volley1.png?raw=true)

从使用者这一侧，我们只需要一个RequestQueue来放入请求，一旦请求放入了队列，首先会判断是否需要缓存，如果说这个请求明确不要缓存，就直接进入NetworkQueue，如果没有明确不缓存，则进入CacheQueue（换句话说就是，非特殊指定的请求默认都会缓存）。CacheDispatcher拿到Request后进入Cache检查，如果未命中Cache，则CacheDispatcher会把这个Request移到NetworkQueue中。NetworkDispatcher则从NetworkQueue中取出Request，然后进行网络请求。两种Dispatcher通过ResponseDelivery将结果传递回Request的回调(Listener)中。

本文尝试把Volley的Dispatcher机制简化，以此学习一个通用的分发器，这是一个典型的生产者消费者问题，生产者和消费者不直接通信，而是通过阻塞队列的中间人，达到生产者和消费者之间解耦。对应上面的图，我们这里介绍的内容就是图中虚线框中的部分。

#### 1.首先我们需要一个队列:

{% highlight java%}
public class Queue {

    // 这个阻塞队列是核心，这里给队列加上了优先级
    private final PriorityBlockingQueue<Task> queue = new PriorityBlockingQueue<Task>(); 
    private DispatchThread[] mDispatchers;

    public Queue(int size) {
        // size就是我们消费者的数量，这里固定为size个，有些系统里可以动态增减
        mDispatchers = new DispatchThread[size]; 
    }

    public void add(Task task) {
        synchronized (queue) {
            queue.add(task);
        }
    }

    public void start() {
        stop();

        for (int i = 0; i < mDispatchers.length; i++) {
            // 将队列传给Worker线程
            DispatchThread thread = new DispatchThread(queue,i); 
            mDispatchers[i] = thread;
            thread.start();
        }

    }

    public void stop() {

    }
}
{% endhighlight %}

这段代码中的queue是我们整个系统的核心部分，所有的其它组件都是围绕他来运作的，线程的阻塞也是通过这个queue里的阻塞特性来实现的。


#### 2. 然后是我们的Worker线程:
{% highlight java%}
public class DispatchThread extends Thread {

    private BlockingQueue<Task> mQueue;
    private int mId;

    public DispatchThread(BlockingQueue<Task> queue, int id) {
        this.mQueue = queue; // 持有队列来获取数据
        mId = id;
    }

    @Override
    public void run() {
        while(true) {

        try {
            // 假如队列为空，则这个线程就把队列阻塞了，其它线程都需要等待。
            Task task = mQueue.take(); 
            System.out.print("Hi, Im thread:" + mId);
            task.hello();
            sleep(500);
            System.out.println("");

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        }

    }
}
{% endhighlight %}


#### 3. 最后就是我们的任务啦:
{% highlight java%}
public class Task implements Comparable{
    private int mId;
    public Task(int i) {
        mId = i;
    }
    public void hello(){
        System.out.println("Hello Im task:" +mId);
    }

    @Override
    public int compareTo(Object o) {
        return 0;
    }
}
{% endhighlight %}

Task 比较简单，可以根据业务改变。

#### 4. 来我们测试一下:
{% highlight java%}
public class Main {

    public static void main(String[] args) {
        Queue queue = new Queue(4);

        for (int i = 0; i < 20; i ++) {
            queue.add(new Task(i));
        }

        queue.start();

        Scanner s = new Scanner(System.in);
        int a;
        while ((a = s.nextInt()) != -1) {
            queue.add(new Task(a));
        }
        System.exit(0);
    }

}
{% endhighlight %}

运行结果：


    Hello Im task:0
    Hello Im task:19
    Hello Im task:18
    Hello Im task:17
    Hello Im task:16
    Hello Im task:15
    Hello Im task:14
    Hello Im task:13
    Hello Im task:12
    Hello Im task:11
    Hello Im task:10
    Hello Im task:9
    Hello Im task:8
    Hello Im task:7
    Hello Im task:5
    Hello Im task:6
    Hello Im task:4
    Hello Im task:3
    Hello Im task:1
    Hello Im task:2



#### 5. 总结

从上面的简化的例子我们可以学习到一个多线程并发的模式，这个模式可以解决大多数并发问题。而通过阻塞而不是轮询，可以降低CPU的消耗。

