---
title: "ThreadLocal"
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

[comment]: <> (# ThreadLocal)

ThreadLocal 是一个线程的本地变量，也就意味着这个变量是线程独有的，是不能与其他线程共享的，这样就可以避免资源竞争带来的多线程的问题，这种解决多线程的安全问题和lock(这里的lock 指通过synchronized 或者Lock 等实现的锁) 是有本质的区别的:

1. lock 的资源是多个线程共享的，所以访问的时候需要加锁。
2. ThreadLocal 是每个线程都有一个副本，是不需要加锁的。
3. lock 是通过时间换空间的做法。
4. ThreadLocal 是典型的通过空间换时间的做法。

当然他们的使用场景也是不同的，关键看你的资源是需要多线程之间共享的还是单线程内部共享的。

ThreadLocal 的使用是非常简单的，看下面的代码：

```java
public class Test {

    public static void main(String[] args) {
        ThreadLocal<String> local = new ThreadLocal<>();
        //设置值
        local.set("hello word");
        //获取刚刚设置的值
        System.out.println(local.get());
    }
}
```

 

## ThreadLocal的数据结构

![img](https://picgo.6and.ltd/img/img_5ff415f5aa909.png)

## 为什么ThreadLocalMap 采用开放地址法来解决哈希冲突?

ThreadLocal 往往存放的数据量不会特别大，这个时候开放地址法简单的结构会显得更省空间（链地址法需要额外的指针空间）

## ThreadLocal应用场景

### 传递参数

[ThreadLocal用于传递参数及优势](https://blog.csdn.net/z394642048/article/details/105382603)

### 保证线程安全

SimpleDateFormat是线程不安全的，例如下面的写法会报错：

日期转换的一个工具类

```
public class DateUtil {

    private static final SimpleDateFormat sdf = 
            new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    public static Date parse(String dateStr) {
        Date date = null;
        try {
            date = sdf.parse(dateStr);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }
}
```

然后将这个工具类用在多线程环境下

```
public static void main(String[] args) {

    ExecutorService service = Executors.newFixedThreadPool(20);

    for (int i = 0; i < 20; i++) {
        service.execute(()->{
            System.out.println(DateUtil.parse("2019-06-01 16:34:30"));
        });
    }
    service.shutdown();
}
```

结果报异常了：

```java
Exception in thread "pool-1-thread-3" Exception in thread "pool-1-thread-12" Exception in thread "pool-1-thread-7" Exception in thread "pool-1-thread-1" Exception in thread "pool-1-thread-11" Exception in thread "pool-1-thread-5" java.lang.NumberFormatException: For input string: ""
  at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
  at java.lang.Long.parseLong(Long.java:601)
  at java.lang.Long.parseLong(Long.java:631)
  at java.text.DigitList.getLong(DigitList.java:195)
  at java.text.DecimalFormat.parse(DecimalFormat.java:2084)
  at java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:1869)
  at java.text.SimpleDateFormat.parse(SimpleDateFormat.java:1514)
  at java.text.DateFormat.parse(DateFormat.java:364)
  at Test.parse(Test.java:15)
  at Test.lambda$main$0(Test.java:27)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
  at java.lang.Thread.run(Thread.java:748)
java.lang.NumberFormatException: multiple points
  at sun.misc.FloatingDecimal.readJavaFormatString(FloatingDecimal.java:1890)
  at sun.misc.FloatingDecimal.parseDouble(FloatingDecimal.java:110)
  at java.lang.Double.parseDouble(Double.java:538)
  at java.text.DigitList.getDouble(DigitList.java:169)
  at java.text.DecimalFormat.parse(DecimalFormat.java:2089)
  at java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:1869)
  at java.text.SimpleDateFormat.parse(SimpleDateFormat.java:1514)
  at java.text.DateFormat.parse(DateFormat.java:364)
  at Test.parse(Test.java:15)
  at Test.lambda$main$0(Test.java:27)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
  at java.lang.Thread.run(Thread.java:748)
java.lang.NumberFormatException: multiple points
  at sun.misc.FloatingDecimal.readJavaFormatString(FloatingDecimal.java:1890)
  at sun.misc.FloatingDecimal.parseDouble(FloatingDecimal.java:110)
  at java.lang.Double.parseDouble(Double.java:538)
  at java.text.DigitList.getDouble(DigitList.java:169)
  at java.text.DecimalFormat.parse(DecimalFormat.java:2089)
  at java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:2162)
  at java.text.SimpleDateFormat.parse(SimpleDateFormat.java:1514)
  at java.text.DateFormat.parse(DateFormat.java:364)
  at Test.parse(Test.java:15)
  at Test.lambda$main$0(Test.java:27)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
  at java.lang.Thread.run(Thread.java:748)
java.lang.NumberFormatException: multiple points
  at sun.misc.FloatingDecimal.readJavaFormatString(FloatingDecimal.java:1890)
  at sun.misc.FloatingDecimal.parseDouble(FloatingDecimal.java:110)
  at java.lang.Double.parseDouble(Double.java:538)
Sat Jun 01 16:34:30 CST 2019
Sat Jun 01 16:34:30 CST 2019
Tue Jun 01 16:34:30 CST 1
Sat Jun 01 16:34:30 CST 2019
Sat Jun 01 16:34:30 CST 2019
Sat Jun 01 16:34:30 CST 2019
Tue Jun 01 16:34:30 CST 1
Sat Jun 01 16:34:30 CST 2019
Sat Jun 01 16:34:30 CST 2019
Sat Jun 01 16:34:30 CST 2019
Sat Jun 01 16:34:30 CST 2019
Sat Jun 01 16:34:30 CST 2019
Sat Jun 01 16:34:30 CST 2019
Sat Jun 01 16:34:30 CST 2019
  at java.text.DigitList.getDouble(DigitList.java:169)
  at java.text.DecimalFormat.parse(DecimalFormat.java:2089)
  at java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:1869)
  at java.text.SimpleDateFormat.parse(SimpleDateFormat.java:1514)
  at java.text.DateFormat.parse(DateFormat.java:364)
  at Test.parse(Test.java:15)
  at Test.lambda$main$0(Test.java:27)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
  at java.lang.Thread.run(Thread.java:748)
java.lang.NumberFormatException: multiple points
  at sun.misc.FloatingDecimal.readJavaFormatString(FloatingDecimal.java:1890)
  at sun.misc.FloatingDecimal.parseDouble(FloatingDecimal.java:110)
  at java.lang.Double.parseDouble(Double.java:538)
  at java.text.DigitList.getDouble(DigitList.java:169)
  at java.text.DecimalFormat.parse(DecimalFormat.java:2089)
  at java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:2162)
  at java.text.SimpleDateFormat.parse(SimpleDateFormat.java:1514)
  at java.text.DateFormat.parse(DateFormat.java:364)
  at Test.parse(Test.java:15)
  at Test.lambda$main$0(Test.java:27)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
  at java.lang.Thread.run(Thread.java:748)
java.lang.NumberFormatException: For input string: "2019."
  at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
  at java.lang.Long.parseLong(Long.java:589)
  at java.lang.Long.parseLong(Long.java:631)
  at java.text.DigitList.getLong(DigitList.java:195)
  at java.text.DecimalFormat.parse(DecimalFormat.java:2084)
  at java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:1869)
  at java.text.SimpleDateFormat.parse(SimpleDateFormat.java:1514)
  at java.text.DateFormat.parse(DateFormat.java:364)
  at Test.parse(Test.java:15)
  at Test.lambda$main$0(Test.java:27)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
  at java.lang.Thread.run(Thread.java:748)

Process finished with exit code 0
```

解决方案4：用ThreadLocal，一个线程一个SimpleDateFormat对象

```java
public class DateUtil {

    private static ThreadLocal<DateFormat> threadLocal = ThreadLocal.withInitial(
            ()-> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));

    public static Date parse(String dateStr) {
        Date date = null;
        try {
            date = threadLocal.get().parse(dateStr);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }
}
```

配合线程池使用，既保证了线程安全，又不至于像 [解决方案1](https://blog.csdn.net/zzti_erlie/article/details/105322946) 创建太多的SimpleDateFormat对象。

[v_tips]ThreadLocal 保证的是单个线程内部访问的是同一个实例，不同线程访问的不是同一个实例。详见：[ThreadLocal线程单例](http://106.55.152.92:30989/archives/1110)[/v_tips]

### 参考链接：

[Java面试必问：ThreadLocal终极篇 淦！](https://www.cnblogs.com/aobing/p/13382184.html)

[被大厂面试官连环炮轰炸的ThreadLocal （吃透源码的每一个细节和设计原理）](https://juejin.cn/post/6844903974454329358)

[万字详解ThreadLocal关键字](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/multi-thread/万字详解ThreadLocal关键字.md)

[面试官：ThreadLocal的应用场景和注意事项有哪些？](https://blog.csdn.net/zzti_erlie/article/details/105322946)

[面试官：ThreadLocal的应用场景和注意事项有哪些？](https://zhuanlan.zhihu.com/p/157690272)
