---
title: "ThreadPoolExecutor"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"Java"
]
series : [
"Manual"
]
images : [
"images/center.png"
]
---

[comment]: <> "# ThreadPoolExecutor"

## 构造方法

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

 

构造函数的参数含义如下：

`corePoolSize`：指定了线程池中的线程数量，它的数量决定了添加的任务是开辟新的线程去执行，还是放到workQueue任务队列中去；

`maximumPoolSize`：指定了线程池中的最大线程数量，这个参数会根据你使用的workQueue任务队列的类型，决定线程池会开辟的最大线程数量；

`keepAliveTime`：当线程池中空闲线程数量超过corePoolSize时，多余的线程会在多长时间内被销毁；

`unit`：keepAliveTime的单位

`workQueue`：任务队列，被添加到线程池中，但尚未被执行的任务；它一般分为直接提交队列、有界任务队列、无界任务队列、优先任务队列几种；

`threadFactory`：线程工厂，用于创建线程，一般用默认即可；

`handler`：拒绝策略；当任务太多来不及处理时，如何拒绝任务；

 

## 阻塞队列

阻塞队列(BlockingQueue)是一个支持两个附加操作的队列。这两个附加的操作是：在队列为空时，获取元素的线程会等待队列变为非空。当队列满时，存储元素的线程会等待队列可用。阻塞队列常用于生产者和消费者的场景，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。阻塞队列就是生产者存放元素的容器，而消费者也只从容器里拿元素。

下图中展示了线程1往阻塞队列中添加元素，而线程2从阻塞队列中移除元素：

![img](http://picgo.6and.ltd/img/img_5ff5cf37d14ce.png)

使用不同的队列可以实现不一样的任务存取策略。在这里，我们可以再介绍下阻塞队列的成员：

![img](http://picgo.6and.ltd/img/img_5ff5cf6113fd2.png)

## 拒绝策略

用户可以通过实现`RejectedExecutionHandler `接口去定制拒绝策略，也可以选择JDK提供的四种已有拒绝策略，其特点如下：

![img](http://picgo.6and.ltd/img/img_5ff5cfc91d801.png)

 

### 参考链接：

[Java线程池实现原理及其在美团业务中的实践](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)

[java线程池ThreadPoolExecutor类使用详解](https://www.cnblogs.com/dafanjoy/p/9729358.html)

[Java多线程-线程池ThreadPoolExecutor构造方法和规则](https://blog.csdn.net/qq_25806863/article/details/71126867)
