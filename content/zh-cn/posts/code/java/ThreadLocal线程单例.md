---
title: "ThreadLocal线程单例"
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

]
---


[comment]: <> (# ThreadLocal线程单例)

`ThreadLocal `保证的是单个线程内部访问的是同一个实例，不同线程访问的不是同一个实例。

```java
package test;

public class Singleton {
    private static final ThreadLocal<Singleton> singleton = new ThreadLocal<Singleton>() {
        @Override
        protected Singleton initialValue() {
            return new Singleton();
        }
    };

    public static Singleton getInstance() {
        return singleton.get();
    }

    private Singleton() {
    }
}
package test;

public class T implements Runnable {
    @Override
    public void run() {
        Singleton instance = Singleton.getInstance();
        System.out.println(instance);
    }
}
```

测试类：

```java
package test;

public class Test {
    public static void main(String[] args){
        System.out.println("main thread "+Singleton.getInstance());
        System.out.println("main thread "+Singleton.getInstance());
        System.out.println("main thread "+Singleton.getInstance());

        Thread thread0 = new Thread(new T());
        Thread thread1 = new Thread(new T());
        thread0.start();
        thread1.start();
    }
}
```

结果：

![img](https://picgo.6and.ltd/img/img_5ffafe534509f.png)

两个线程（线程0和线程1）拿到的对象并不是同一个对象，但是同一线程能保证拿到的是同一个对象，即线程单例。
ThreadLocal是基于ThreadLocalMap来实现的，所以我们在调用get方法的时候，默认走的就是这个map，不用指定key，它维持了线程间的隔离。
ThreadLocal隔离了多个线程对数据的访问冲突。对多线程资源共享的问题，假如使用的是同步锁，那么就是以时间换空间的方式；那假如使用ThreadLocal，那就是用空间换时间的方式。

### 参考链接：

[“单例”模式-ThreadLocal线程单例](https://www.jianshu.com/p/1aff161075c7)
