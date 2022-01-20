---
title: "Java堆外内存溢出"
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

[comment]: <> "# Java堆外内存溢出"

前段时间在做一个实时人脸抓拍项目的时候，遇到了一个堆外内存OOM的问题，现在把思路好好整理一下。

项目中用opencv通过rtsp协议，实时的读取通用网络摄像头的视频帧。因为项目中多处用到了org.opencv.core.Mat这个对象，而Mat对象的构造是通过调用native方法实现的，也就是说构造Mat对象的时候，会在堆外分配内存：

```c++
    //
    // C++: Mat::Mat()
    //

    // javadoc: Mat::Mat()
    public Mat()
    {

        nativeObj = n_Mat();

        return;
    }
    // C++: Mat::Mat()
    private static native long n_Mat();
```

堆外分配的内存不受JVM的内存管理。由于又没有主动调用Mat.realse()去释放堆外内存，导致堆外内存OOM。

其实解决的办法很简单，可以在Mat对象使用完毕后直接调用Mat.realse()释放堆外内存。（没有试过，本人使用的下面的方式）

但是，Mat对象充斥着整个项目，要跟踪Mat对象的生命周期显得有点复杂，而且因为太多地方使用了Mat对象，很有可能遗漏调用Mat.realse()释放内存。因此，还是想把这部分内存的释放交由JVM来做，具体的方式是：定期的调用System.gc()执行垃圾回收（很多人说System.gc()只是建议JVM执行垃圾回收，并不是命令，是否执行取决去JVM自己，但是，经我实测，每次调用System.gc()都会触发垃圾回收。），JVM在垃圾回收前会执行每个**空java对象（null）**的finalize()方法，而Mat对象的finalize()方法正好实现了释放内存的逻辑：

```java
    @Override
    protected void finalize() throws Throwable {
        n_delete(nativeObj);
        super.finalize();
    }
    // native support for java finalize()
    private static native void n_delete(long nativeObj);
```

因为会定时的调用System.gc()触发Full GC, 而Full GC的之前会调用那些不再被引用的Mat对象的finalize()方法释放它的堆外内存，所以间接的实现了由JVM释放堆外内存的目的。

但是，这种做法并不好，因为通过System.gc()强制定期执行Full GC，势必会影响java应用本身。

为何一定要复制到DirectByteBuffer来读写（系统调用）？

GC会回收无用对象，同时还会进行碎片整理，移动对象在内存中的位置，来减少内存碎片。DirectByteBuffer不受GC控制。如果不用DirectByteBuffer而是用HeapByteBuffer，如果在调用系统调用时，发生了GC，导致HeapByteBuffer内存位置发生了变化，但是内核态并不能感知到这个变化导致系统调用读取或者写入错误的数据。所以一定要通过不受GC影响的DirectByteBuffer来进行IO系统调用。

假设我们要从网络中读入一段数据，再把这段数据发送出去的话，采用Non-direct ByteBuffer的流程是这样的：

```
网络 –> 临时的DirectByteBuffer –> 应用 Non-direct ByteBuffer –> 临时的Direct ByteBuffer –> 网络
```

这种方式是直接在堆外分配一个内存(即，native memory)来存储数据，
程序通过JNI直接将数据读/写到堆外内存中。因为数据直接写入到了堆外内存中，所以这种方式就不会再在JVM管控的堆内再分配内存来存储数据了，也就不存在堆内内存和堆外内存数据拷贝的操作了。这样在进行I/O操作时，只需要将这个堆外内存地址传给JNI的I/O的函数就好了。

采用Direct ByteBuffer的流程是这样的：

```
网络 –> 应用 Direct ByteBuffer –> 网络
```



#### 参考：

[JDK核心JAVA源码解析（4） - 堆外内存、零拷贝、DirectByteBuffer以及针对于NIO中的FileChannel的思考](https://blog.csdn.net/zhxdick/article/details/81084672)

[Java 内存之直接内存（堆外内存）](https://www.kancloud.cn/zhangchio/springboot/806316)

[10 双刃剑：合理管理 Netty 堆外内存](http://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/Netty%20%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86%E5%89%96%E6%9E%90%E4%B8%8E%20RPC%20%E5%AE%9E%E8%B7%B5-%E5%AE%8C/10%20%20%E5%8F%8C%E5%88%83%E5%89%91%EF%BC%9A%E5%90%88%E7%90%86%E7%AE%A1%E7%90%86%20Netty%20%E5%A0%86%E5%A4%96%E5%86%85%E5%AD%98.md)

[Java直接内存是属于内核态还是用户态？](https://www.zhihu.com/question/376317973)

[堆外内存 之 DirectByteBuffer 详解](https://www.jianshu.com/p/007052ee3773)
