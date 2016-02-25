---
layout: post
title: Java多线程分发器实例
description: "在Android的网络框架Volley中使用了多线程分发网络请求的方法, 这种方法可以看作是一个生产者消费者问题, 让我们也跟着来实现一个这样的小分发器"
category: Java
tags: ["Java", "Android", "Volley", "多线程"]
---



#### 1.首先我们需要一个队列:

{% highlight java%}
public class Queue {

    private final PriorityBlockingQueue<Task> queue = new PriorityBlockingQueue<Task>();
    private DispatchThread[] mDispatchers;


    public Queue(int size) {
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
            DispatchThread thread = new DispatchThread(queue,i);
            mDispatchers[i] = thread;
            thread.start();
        }

    }

    public void stop() {

    }
}
{% endhighlight %}

#### 2. 然后是我们的Worker线程:
{% highlight java%}
public class DispatchThread extends Thread {

    private BlockingQueue<Task> mQueue;
    private int mId;

    public DispatchThread(BlockingQueue<Task> queue, int id) {
        this.mQueue = queue;
        mId = id;
    }

    @Override
    public void run() {
        while(true) {

        try {
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