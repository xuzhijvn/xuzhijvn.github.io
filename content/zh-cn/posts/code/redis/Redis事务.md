+++
title = "Redis事务"
description = ""
date = 2021-10-08T16:45:53+08:00
featured = false
comment = true
toc = true
reward = true
categories = [
  "编程思想"
]
tags = [
  "Redis"
]
series = ["Manual"]
images = []

+++

<!--more-->

redis它只是做到了：

1. 它认为的原子性。（单个命令原子，多条命令不一定原子）
2. 隔离性。（单线程执行，各事务串行执行，天然满足隔离）
3. AOF/RDB保证了部分的持久性。（持久化的时候可能存在数据丢失）
4. 它不存在ACID中的C的概念，因为它没有约束。

## 1. 命令使用错误

这种错误redis在执行前就能检查出来，因此整个事务都不执行。

```sh
127.0.0.1:6379> get name
"xumeili"
127.0.0.1:6379> get age
"28"
127.0.0.1:6379> get sex
"female"
127.0.0.1:6379> multi 
OK
127.0.0.1:6379> set name xuzhijun
QUEUED
127.0.0.1:6379> set age
(error) ERR wrong number of arguments for 'set' command
127.0.0.1:6379> set sex male
QUEUED
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get name
"xumeili"
127.0.0.1:6379> get age
"28"
127.0.0.1:6379> get sex
"female"
```

## 2. 命令执行错误

这种错误redis无法在执行前发现，因此不影响这种错误的前后命令被提交。

```sh
127.0.0.1:6379> get name
"xumeili"
127.0.0.1:6379> get age
"28"
127.0.0.1:6379> get sex
"female"
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set name tony
QUEUED
127.0.0.1:6379> sadd age 100
QUEUED
127.0.0.1:6379> set sex male
QUEUED
127.0.0.1:6379> exec
1) OK
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
3) OK
127.0.0.1:6379> get name
"tony"
127.0.0.1:6379> get age
"28"
127.0.0.1:6379> get sex
"male"
```

## 3. 使用watch

事务内key值发生变化，整个事务不提交。

session1:

```sh
127.0.0.1:6379> get name
"xuzhijun"
127.0.0.1:6379> get age
"300"
127.0.0.1:6379> get sex
"female"
127.0.0.1:6379> watch age
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set name tony
QUEUED
127.0.0.1:6379> set age 100
QUEUED
127.0.0.1:6379> set sex male
QUEUED
127.0.0.1:6379> exec
(nil)
127.0.0.1:6379> get name
"xuzhijun"
127.0.0.1:6379> get age
"28"
127.0.0.1:6379> get sex
"female"
```

session2:

```sh
127.0.0.1:6379> set age 28
OK
127.0.0.1:6379> get age
"28"
```

## 参考

[Redis 设计与实现](https://redisbook.readthedocs.io/en/latest/feature/transaction.html)

[redis事务一致性问题？？？](https://www.zhihu.com/question/60189169)

[如何理解数据库事务中的一致性的概念？](https://www.zhihu.com/question/31346392)
