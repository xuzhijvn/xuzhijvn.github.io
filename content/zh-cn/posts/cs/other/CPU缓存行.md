---
title: "CPU缓存行"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"计算机科学"
]
tags : [
"CS"
]
series : [
"Manual"
]
images : [

]
---

[comment]: <> "# CPU缓存行"

[JAVA 拾遗 — CPU Cache 与缓存行](https://www.cnkirito.moe/cache-line/)

[既然CPU有缓存一致性协议（MESI），为什么JMM还需要volatile关键字？](https://www.zhihu.com/question/296949412)

简而言之，CPU里的缓存，buffer，queue有很多种。MESI只能在一种情况下解决核心专有Cache之间不一致的问题。

此外，如果有些CPU不支持MESI协议，那么必须用其他办法来实现等价的效果，比如总是用锁总线的方式，或者明确的fence指令来保证volatile想达到的目标。

如果CPU是单核心的，cache是专供这个核心的，MESI理论上也就没有用了。但是依然要考虑主存和Cache被多个线程切换访问时带来的不一致问题。

总之，volatile是一个高层的表达意图的“抽象”，而MESI是为了实现这个抽象，在某种特定情况下需要使用的一个实现细节。

[CPU缓存行](https://www.jianshu.com/p/e338b550850f)

[Java内存访问重排序的研究](https://tech.meituan.com/2014/09/23/java-memory-reordering.html)

