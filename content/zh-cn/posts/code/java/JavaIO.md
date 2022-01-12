+++
title = "JavaIO"
description = ""
date = 2021-11-12T17:40:13+08:00
featured = false
comment = true
toc = true
reward = true
categories = [
  ""
]
tags = [
  ""
]
series = []
images = []

+++

从操作系统层面划分，多采用史蒂夫-理查德在《The Sockets Networking API》：阻塞io、非阻塞io、io多路复用、信号驱动io、异步io


从Java层面去划分：bio, nio, aio, 分别对应操作系统的阻塞io、io多路复用、异步io

<!--more-->

## bio

`特点`：数据从 网卡->内核，内核->用户空间 两阶段都是阻塞的；每一个客户端请求都需要一个线程去处理网络IO。

`优点`：io模型简单，不容易出错

`场景`：并发量不大的web应用，例如：tomcat，bio+线程池（伪异步I/O编程）

`缺点`：难以应对高并发的场景

> read/recvfrom

## nio

`特点`：把多个I/O的阻塞复用到同一个select/poll/epoll的阻塞上，从而使得在单线程的情况下可以同时处理多个客户端请求。

`优点`：更小的系统开销，处理更多的连接。

`场景`：高并发web应用，长连接短数据（即时通讯）

`缺点`：连接数不是很高的话，nio不一定比线程池+bio性能更好，可能延迟还更大，因为同样是一次io, nio要发生select+recvfrom两次系统调用，bio只用调一次recvfrom。

> select 多路复用
>
> poll 多路复用（没有最大文件描述符数量的限制）
>
> epoll 基于事件驱动的多路复用

## aio

`特点`：不像nio那样需要遍历连接，aio的读写操作都是异步的，应用发起aio_read/aio_write操作后就不用管了，数据准备好了自动回调。

`优点`：并发性高、CPU利用率高、线程利用率高（因为它连select遍历线程都不需要）

`场景`：高并发

`缺点`：不适合轻量级数据传输，因为进程之间频繁的通信在追错、管理和资源消耗上不是很可观。

> Java 通过AsynchronousServerSocketChannel、AsynchronousSocketChannel支持aio编程
>
> windows: IOCP(I/O Completion Port，I/O完成端口)
>
> Linux: 不管是Linux的AIO原生库、Glibc的AIO库、最新版的内核的IO_URING等等，都不可能是完全的AIO
>
> Java Linux AIO是通过选择器+线程池，也即Epoll + ThreadPoolExecutor模拟的



>
> read/wirte是通用的文件描述符操作；recv/send 通常应用于TCP；recvfrom/sendto通常应用于UDP。 



## 参考

[Java AIO 网络编程真相](https://zhuanlan.zhihu.com/p/372624299)

[深入理解Java I/O模型](https://juejin.cn/post/6844903839439519758)

[什么是 NIO ？6000 字详解 NIO](https://xie.infoq.cn/article/fb524c4992beea6bb4487af87)

