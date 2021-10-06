---
title: "Java协程"
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

[comment]: <> "# Java协程"

## 1. 现状

时至今日，JDK17已经正式发布，Java也没有在语言层面对协程提供原生支持。

一定要在Java中使用协程的话，可以使用诸如[quasar](https://github.com/puniverse/quasar), [kilim](https://github.com/kilim/kilim), [coroutines](https://github.com/offbynull/coroutines) 第三方库感受一下，它们的原理基本都是字节码增强 + Java Agent机制，这种做法一是对性能影响很大，对JIT编译器的影响也非常大，另外这些库都几年前就不在更新了，远达不到生产使用的标准。

还可以使用Kotlin混合编程，Kotlin中的协程本质上还是一套基于原生Java Thread API 的封装，和Go中的协程完全不是一个东西，不要混淆，更谈不上什么性能更好，Kotlin中的协程最大的价值是写起来比RxJava的线程切换还要方便，几乎就是用阻塞的写法来完成非阻塞的任务。

openjdk也有正在孵化的官方协程项目[loom](https://github.com/openjdk/loom)，其能否release还需拭目以待。

## 2. 成因

是什么原因导致Oracle一直不着急推出对协程的支持呢？

先返回到问题的本源。当我们希望引入协程，我们想解决什么问题。我想不外乎下面几点：

1. 节省资源：节省内存、节省分配线程的开销（创建和销毁线程要各做一次syscall）、节省大量线程切换带来的开销

3. 与NIO配合实现非阻塞的编程，提高系统的吞吐

4. 使用起来更加舒服顺畅，同步的编程风格编写异步程序

### 2.1 节省资源

#### 1. 节省内存

我们以常见的Java Web举例，spingboot分配给tomcat的线程池大小默认值是200，即使按照1M线程大小计算，200M的内存占用对于动辄几个G的Java Web应用并不算什么。

即使是IM的场景，有数百万的长链接需要维护，也可以使用NIO+Worker线程应对。

还可以调整线程栈占用内存大小（`-Xss1024k` 或者  `-XX:ThreadStackSize=1024k`）

#### 2. 节省线程分配开销

线程池

#### 3. 节省线程切换开销

我们仍然以Java Web举例，大量的线程大部分时间实际上因为IO（发请求/读DB）而挂起，根本不会参与OS的线程切换，现实当中一个最大200线程的服务器可能同一时刻的“活跃线程”总数只有数十而已。其开销没有想象的那么大。

### 2.2 非阻塞编程

nio 👉 netty

### 2.3 优雅异步编程

响应式编程库

可见Java对于引入协程机制并不那么紧迫；并且Java不像Golang等新语言，没有历史包袱，它们可以不提供线程只提供协程编程，但是Java如果没有thread，也没有ThreadLocal，@Transactional不起作用了，又没有等价的工具，是不是很郁闷？

## 参考

[为什么 Java 坚持多线程不选择协程？](https://www.zhihu.com/question/332042250)

[Kotlin 协程真的比 Java 线程更高效吗？](https://segmentfault.com/a/1190000021548651)

[硬核系列 | 深入剖析 Java 协程](https://xie.infoq.cn/article/cef6d2931a54f85142d863db7)

[Golang 的 协程调度机制 与 GOMAXPROCS 性能调优](https://juejin.cn/post/6844903662553137165)

