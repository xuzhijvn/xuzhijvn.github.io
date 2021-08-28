---
title: "Redis配置指南"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"Redis"
]
series : [
"Manual"
]
images : [
"images/center.png"
]
---

[comment]: <> (# Redis配置指南)

## 1. 安装

```shell
$ wget http://download.redis.io/releases/redis-5.0.8.tar.gz
$ tar xzf redis-5.0.8.tar.gz
$ cd redis-5.0.8
$ make
```

## 2.配置

1. 注释掉bind 127.0.0.1
2. protected-mode yes
3. requirepass xxxpassword
4. daemonize yes

## 3. 启动

```shell
cd src
./redis-server
```

或者

```shell
cd src
./redis-server ../redis.conf
```

## 4. 停止

```shell
cd src
./redis-cli
auth xxxpassword
shutdown
exit
```

## 5. 卸载

```shell
find / -name "redis*" | xargs rm -rf
```

## 6.远程连接

window连接远程redis:

```shell
redis-cli -h 193.112.37.xxx -p 6379 -a xxxpassword
```

