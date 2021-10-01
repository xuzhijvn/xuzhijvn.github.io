+++
title = "RPC综述"
description = ""
date = 2021-09-30T16:20:07+08:00
featured = false
comment = true
toc = true
reward = true
categories = [
  "架构演进"
]
tags = [
  "SA"
]
series = ["Manual"]
images = []

+++

RPC的本质是要使得微服务远程调用像调用本地方法一样无感。

<!--more-->

目前微服务远程调用大致分成三种实现方案：

1. 纯http调用；

   `通用`

2. 以Sping Cloud Feign的为代表的底层http调用方式；（泛化调用？）

    `无感`

   `http传输 -> 冗余数据-> 传输效率低`，`更具通用性，服务提供方与具体开发语言无关`

   `文本序列化 -> 序列化后字节占用大 & 耗时久 ` ，`可读性强`

   Feign使用JDK动态代理等设计屏蔽了纯http的调用细节，使得远程调用像使用本地方法一样无感，序列化默认使用Jackson，feign.Client有四种：

   （1）Client.Default类：默认的feign.Client 客户端实现类，内部使用HttpURLConnnection 完成URL请求处理；

   （2）ApacheHttpClient 类：内部使用 Apache httpclient 开源组件完成URL请求处理的feign.Client 客户端实现类；

   （3）OkHttpClient类：内部使用 OkHttp3 开源组件完成URL请求处理的feign.Client 客户端实现类。

   （4）LoadBalancerFeignClient 类：内部使用 Ribben 负载均衡技术完成URL请求处理的feign.Client 客户端实现类

3. 以dubbo未代表的自己实现一套RPC方案，方案包括动态代理、序列化/反序列化、协议约定、网络传输；

    `无感`

   `tcp传输 -> 为rpc专门设计 -> 传输效率高`，`不通用性，服务调用方、服务提供方都必须是dubbo服务`

   `二进制序列化 -> 序列化后字节占有小 & 耗时短 `，`可读性差，不利于调试`

   当然，除了默认的dubbo RPC（二进制序列化 + tcp协议），dubbo也支持 `http://`（二进制序列化 + http协议）、`hessian://`（二进制序列化 + http协议）、`webservices://` （文本序列化 + http协议）、`rest://`（文本序列化 + http协议）。上述讨论dubbo rpc优缺点是基于默认 `dubbo://` 协议

> [dubbo:// 协议特性](https://dubbo.apache.org/zh/docsv2.7/user/references/protocol/dubbo/#特性)
>
> 缺省协议，使用基于 netty `3.2.5.Final` 和 hessian2 `3.2.1-fixed-2(Alibaba embed version)` 的 tbremoting 交互。
>
> - 连接个数：单连接
> - 连接方式：长连接
> - 传输协议：TCP
> - 传输方式：NIO 异步传输
> - 序列化：Hessian 二进制序列化
> - 适用范围：传入传出参数数据包较小（建议小于100K），消费者比提供者个数多，单一消费者无法压满提供者，尽量不要用 dubbo 协议传输大文件或超大字符串。
> - 适用场景：常规远程服务方法调用



设计一个rpc框架大概需要：

<img src="https://picgo.6and.ltd/img/4213082d93da0d63cb78420e447ad000.png" alt="设计一个rpc框架" style="zoom: 67%;" />

## 参考

[RPC 核心，万变不离其宗](https://xie.infoq.cn/article/7895a07a4b32034e0b4a2b5b5)

[dubbo 协议](https://dubbo.apache.org/zh/docsv2.7/user/references/protocol/dubbo/)

[dubbo-go 中 REST 协议实现](https://dubbo.apache.org/zh/blog/2021/01/14/dubbo-go-%E4%B8%AD-rest-%E5%8D%8F%E8%AE%AE%E5%AE%9E%E7%8E%B0/)

[开发 REST 应用](https://dubbo.apache.org/zh/docsv2.7/user/rest/#%E5%AE%9A%E5%88%B6%E5%BA%8F%E5%88%97%E5%8C%96)

[xml、json、protobuf序列化协议](https://zhuanlan.zhihu.com/p/91313277)

[SpringCloud 中 Feign 核心原理](https://www.cnblogs.com/crazymakercircle/p/11965726.html)

[Dubbo 中的 http 协议](https://www.cnkirito.moe/dubbo-http-protocol/)

[你应该知道的RPC原理](https://www.cnblogs.com/LBSer/p/4853234.html)

