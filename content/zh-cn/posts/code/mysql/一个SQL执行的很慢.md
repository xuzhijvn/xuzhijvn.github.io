---
title: "一个SQL执行的很慢"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"MySQL"
]
series : [
"Manual"
]
images : [
"images/center.png"
]
---

[comment]: <> "# 一个SQL执行的很慢"

一个SQL执行的很慢，我们要分两种情况讨论：

## 大多数情况下很正常，偶尔很慢

- 数据库在刷新脏页，例如 redo log 写满了需要同步到磁盘。

当我们要往数据库插入一条数据、或者要更新一条数据的时候，我们知道数据库会在内存中把对应字段的数据更新了，但是更新之后，这些更新的字段并不会马上同步持久化到磁盘中去，而是把这些更新的记录写入到 redo log 日记中去，等到空闲的时候，在通过 redo log 里的日记把最新的数据同步到磁盘中去。`写redo log是顺序io`

- 执行的时候，遇到锁，如表锁、行锁。

## 这条 SQL 语句一直执行的很慢

- 没有用上索引：例如该字段没有索引；由于对字段进行运算、函数操作导致无法用索引。
- 数据库选错了索引。

## 参考

[腾讯面试：一条SQL语句执行得很慢的原因有哪些？---不看后悔系列](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485185&idx=1&sn=66ef08b4ab6af5757792223a83fc0d45&chksm=cea248caf9d5c1dc72ec8a281ec16aa3ec3e8066dbb252e27362438a26c33fbe842b0e0adf47&token=79317275&lang=zh_CN#rd)
