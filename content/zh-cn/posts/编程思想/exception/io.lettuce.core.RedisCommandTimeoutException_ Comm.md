---
title: "io.lettuce.core.RedisCommandTimeoutException: Command timed out after 5 second(s)"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"redis"
]
series : [
"Manual"
]
images : [
"images/center.png"
]
---

[comment]: <> (# io.lettuce.core.RedisCommandTimeoutException: Command timed out after 5 second&#40;s&#41;)

lettuce连接redis报错`io.lettuce.core.RedisCommandTimeoutException: Command timed out after 5 second(s)`，我的spring.redis.timeout = 5000。

解决办法：

1、登陆redis容器
2、输入redis-cli进入redis控制台
3、设置 `CONFIG SET timeout "60"`
4、设置 `CONFIG SET tcp-keepalive "300"`

参考链接：[Redis 配置](https://www.runoob.com/redis/redis-conf.html)

