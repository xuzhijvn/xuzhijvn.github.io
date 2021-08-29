+++
title = "Nginx负载均衡"
description = ""
date = 2021-08-29T23:29:12+08:00
comment = true
toc = true
reward = true
categories = [
"编程思想"
]
tags =[
"nginx"
]
series = [
"Manual"
]
images = [
"images/center.png"
]

+++

<!--more-->

Nginx 实现负载均衡的分配策略有很多，Nginx 的 upstream 目前支持以下几种方式：

- 轮询（默认）：每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。
- weight：指定轮询几率，weight 和访问比率成正比，用于后端服务器性能不均的情况。
- ip_hash：每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决 session 的问题。
- fair（第三方）：按后端服务器的响应时间来分配请求，响应时间短的优先分配。
- url_hash（第三方）：按访问 url 的 hash 结果来分配请求，使每个 url 定向到同一个后端服务器，后端服务器为缓存时比较有效。

## 参考

[LVS、Nginx、HAProxy、keepalive 的工作原理](https://www.jianshu.com/p/16e9c84fdb3c)

