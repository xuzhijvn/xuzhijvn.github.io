---
title: "API接口安全设计"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"架构演进"
]
tags : [
"SA"
]
series : [
"Manual"
]
images : [
"images/center.png"
]
---

[comment]: <> "# API接口安全设计"

1. `验签`：防接口被肆意调用

client对request签名，server对请求进行验签。

2. `鉴权`：防调用没有权限的接口

token鉴权，实际上验签、鉴权通常是一同处理的，例如：OAuth2

3. `验证请求的timestamp`：防盗链

当前时间 - timestamp > 阈值，则说明该请求已经失效。

4. `nonce随机数`：防重复提交

上述验证timestamp的过程中存在一个问题，例如：阈值 = 60s，那么60s内发生重复提交怎么办？

解决办法是：server端保存带随机数的请求（随机数+原请求 = 新请求，通常是redis保存）。

5. `接口幂等性保证`（唯一主键、乐观锁）

幂等性：以相同的请求调用这个接口一次和调用这个接口多次，对系统产生的影响是相同的。

[高并发下接口幂等性解决方案](https://www.cnblogs.com/linjiqin/p/9678022.html)

恶意的重复提交：请求的timestamp、nonce较上一次没变，这种情况通过redis保存带随机数的请求可以杜绝。

用户误操作导致的重复提交：前端控制+幂等设计。

幂等与重复提交的区别在于：幂等是在重复提交已经发生了的情况下，如何保证多次调用接口的结果一致，重复提交在前，幂等保证在后。

### 参考链接：

[如何保证接口的幂等性](https://segmentfault.com/a/1190000020172463)

[说说API的防重放机制](https://www.cnblogs.com/yjf512/p/6590890.html)

[Java生鲜电商平台-API接口设计之token、timestamp、sign 具体架构与实现（APP/小程序，传输安全）](https://www.cnblogs.com/jurendage/p/12653865.html)

[阿里一面：如何保证API接口数据安全？](https://mp.weixin.qq.com/s/p01MF3hA8vluVn53YltEOA)

[实现接口幂等性的几种方案](https://www.cnblogs.com/54chensongxia/p/12598944.html)
