---
title: "HashMap和HashTable区别"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"Java"
]
series : [
"Manual"
]
images : [

]
---

[comment]: <> "# HashMap和HashTable区别"

1. HashMap是HashTable的轻量级版本， HashTable是线程安全的，其方法都被synchronized关键同步
2. HashMap 把 Hashtable 的 contains 方法去掉了，改成 containsValue 和 containsKey。因为 contains 方法容易让人引起误解。
3. HashMap允许将 null 作为一个 entry 的 key 或者 value，而 Hashtable 不允许。
4. HashTable 继承自 Dictionary 类，而 HashMap 是 Java1.2 引进的 Map interface 的一个实现。
5. Hashtable 和 HashMap 采用的 hash/rehash 算法都大概一样，所以性能不会有很大的差异。
