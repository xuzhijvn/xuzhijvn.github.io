+++
title = "volatile"
description = ""
date = 2021-09-13T15:37:40+08:00
featured = false
comment = true
toc = true
reward = true
categories = [
"编程思想"
]
tags = [
"Java"
]
series = [
"Manual"
]
images = [
]

+++

JVM volatile用于保证程序`可见性`、`顺序性`，但是不保证`原子性`。volatile实现原理是通过在操作变量之前，多加一个`lock前缀指令`，通过汇编可以看到这个前缀指令。

当谈到顺序性时常会提到`内存屏障`，常见的硬件层面`内存屏障`有：`sfence`    `lfence`     `mfence` ，`lock前缀指令`不是内存屏障，而是一种锁，执行时会锁住内存子系统来确保执行顺序，甚至跨多个CPU，JVM利用lock前缀指令的特点实现了`可见性` 和 `顺序性`，lock前缀指令实现可见性比较好理解，主要是利用CPU提供的缓存一致性协议（例如Intel的MESI）,当然更差一点的还有lock总线的方式（限制CPU访问内存）。

JMM层面为了实现`顺序性`，又抽象出四个`内存屏障`的概念：`LoadLoad`     `StoreStore`    `LoadStore`    `StoreLoad`，字节码层面并没有`内存屏障`的指令，JVM的C++代码会有四个同名函数与之对应，JVM遇到volatile变量便会在其前后执行对应的函数，从而实现内存屏障，具体来说：

```java
LoadLoadBarrier
volatile 读操作
LoadStoreBarrier

StoreStoreBarrier
volatile 写操作
StoreLoadBarrie
```



<!--more-->

上面简单总结了下`可见性`、`顺序性` 的原理，详情🔎可见本文参考链接，这里不在赘述，下面主要总结下，为什么说volatile不保证原子性，以及volatile的应用场景。

### 原子性

下面段代码的结果有可能不是 100000，有可能小于 100000。

```java
public class VolatileTest implements Runnable {

    public static volatile int num;

    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            num++;
        }

    }

    public static void main(String[] args) {
        for(int i = 0; i < 100; i++) {
            VolatileTest t = new VolatileTest();
            Thread t0 = new Thread(t);
            t0.start();
        }
        System.out.println(num);
        
    }
}
```

因为 num++ 并不是原子性的，一个num++ 语句其实包括3个操作：

1. 获取i
2. i自增
3. 回写i

num++ 本身不是原子的，也不会因为加了volatile就变成原子操作。

### 应用场景

#### 状态标记量

```java
volatile boolean flag = false;
 
while(!flag){
    doSomething();
}
 
public void setFlag() {
    flag = true;
}
```

#### 单例双重校验

```java
class Singleton{
    private volatile static Singleton instance = null;
     
    private Singleton() {
         
    }
     
    public static Singleton getInstance() {
        if(instance==null) {
            synchronized (Singleton.class) {
                if(instance==null)
                    instance = new Singleton();
            }
        }
        return instance;
    }
}
```

### 总结

volatile关键字：

1. 它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成；
2. 它会强制将对缓存的修改操作立即写入主存；
3. 如果是写操作，它会导致其他CPU中对应的缓存行无效。

## 参考

[面试官：你说你精通Java并发，给我讲讲 volatile](https://mp.weixin.qq.com/s/3AtT558QdHGI6Kr2H9wv4w)

[volatile底层原理详解](https://zhuanlan.zhihu.com/p/133851347)

[Java并发编程：volatile关键字解析](https://www.cnblogs.com/dolphin0520/p/3920373.html)

[volatile与lock前缀指令](https://blog.csdn.net/qq_26222859/article/details/52235930)

