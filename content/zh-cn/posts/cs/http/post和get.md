---
title: "POST和GET"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"计算机科学"
]
tags : [
"CS","http"
]
series : [
"Manual"
]
images : [
"images/center.png"
]
---

[comment]: <> "# POST和GET"

## RFC标准区别

根据RFC定义的http标准，区别如下：

- GET 用于获取信息，是无副作用的，是幂等的，且可缓存
- POST 用于修改服务器上的数据，有副作用，非幂等，不可缓存

## 实际区别

GET 和 POST 方法没有实质区别，只是报文格式不同。

GET 和 POST 只是 HTTP 协议中两种请求方式，而 HTTP 协议是基于 TCP/IP 的应用层协议，无论 GET 还是 POST，用的都是同一个传输层协议，所以在传输上，没有区别。

你可以在GET请求的body里面发参数，也可以在POST请求的url后面跟参数。

现实中，之所以会有诸如：GET请求有长度限制这样的差异，是与浏览器和服务端实现细节有关，与http协议本身无关，不是http协议规定的这些差异。

## 其他区别

有些文章中提到，post 会将 header 和 body 分开发送，先发送 header，服务端返回 100 状态码再发送 body。

HTTP 协议中没有明确说明 POST 会产生两个 TCP 数据包，而且实际测试(Chrome)发现，header 和 body 不会分开发送。

所以，header 和 body 分开发送是部分浏览器或框架的请求方法，不属于 post 必然行为。
