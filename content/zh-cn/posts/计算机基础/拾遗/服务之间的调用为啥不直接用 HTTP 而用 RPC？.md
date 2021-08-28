
---
title: "服务之间的调用为啥不直接用 HTTP 而用 RPC？"
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

[comment]: <> (# 服务之间的调用为啥不直接用 HTTP 而用 RPC？)



1. 使用自定义 TCP 协议进行传输就会避免上面这个问题，极大地减轻了传输数据的开销。 这也就是为什么通常会采用自定义 TCP 协议的 RPC 来进行进行服务调用的真正原因。
2. 除此之外，成熟的 RPC 框架还提供好了“服务自动注册与发现”、"智能负载均衡"、“可视化的服务治理和运维”、“运行期流量调度”等等功能，这些也算是选择 RPC 进行服务注册和发现的一方面原因吧！
