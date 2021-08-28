
---
title: "http2"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"计算机基础"
]
tags : [
"CS"
]
series : [
"Manual"
]
images : [
"images/center.png"
]
---

[comment]: <> (# http2)

http/2在http/1系列的基础上优化了通信效率，主要得益于如下几点改进：

## 一、 多路复用的单一长连接

### 1.单一长连接

在HTTP/2中，客户端向某个域名的服务器请求页面的过程中，只会创建一条TCP连接，即使这页面可能包含上百个资源。 单一的连接应该是HTTP2的主要优势，单一的连接能减少TCP握手（还有ssl握手的开销）带来的时延 。HTTP2中用一条单一的长连接，避免了创建多个TCP连接带来的网络开销，提高了吞吐量。

### 2.多路复用

HTTP2虽然只有一条TCP连接，但是在逻辑上分成了很多stream。HTTP2把要传输的信息分割成一个个二进制帧，首部信息会被封装到HEADER Frame，相应的request body就放到DATA Frame,一个帧你可以看成路上的一辆车,只要给这些车编号，让1号车都走1号门出，2号车都走2号门出，就把不同的http请求或者响应区分开来了。但是，这里要求同一个请求或者响应的帧必须是有有序的，要保证FIFO的，但是不同的请求或者响应帧可以互相穿插。这就是HTTP2的多路复用，是不是充分利用了网络带宽，是不是提高了并发度？

## 二、头部压缩和二进制格式

http1.x一直都是plain text,对此我只能想到一个优点，便于阅读和debug。但是，现在很多都走https，SSL也把plain text变成了二进制，那这个优点也没了。 于是HTTP2搞了个HPACK压缩来压缩头部，减少报文大小(调试这样的协议将需要curl这样的工具，要进一步地分析网络数据流需要类似Wireshark的http2解析器)。

## 三、服务端推送Server Push

这个功能通常被称作“缓存推送”。主要的思想是：当一个客户端请求资源X，而服务器知道它很可能也需要资源Z的情况下，服务器可以在客户端发送请求前，主动将资源Z推送给客户端。这个功能帮助客户端将Z放进缓存以备将来之需。

## 总结

1. 单一的长连接，减少了SSL握手的开销
2. 头部被压缩，减少了数据传输量
3. 多路复用能大幅提高传输效率，不用等待上一个请求的响应
4. 不用像http1.x那样把多个文件或者资源弄成一个文件或者资源（http1.x常见的优化手段），这时候，缓存就能更容易命中啊（http1.x里面你揉成一团的东西怎么命中缓存？）

### 参考

[HTTP/2 相比 1.0 有哪些重大改进？](https://www.zhihu.com/question/34074946?utm_source=wechat_session&utm_medium=social&s_s_i=U0%2FPUrgIwamqLmBabvN4DxqQrrMCcV%2BFEUIvuj5nLm4%3D&s_r=1)

[HTTP 2 的新特性你 get 了吗？](https://cloud.tencent.com/developer/article/1004874)

[HTTP/2 服务器推送（Server Push）教程](http://www.ruanyifeng.com/blog/2018/03/http2_server_push.html)
