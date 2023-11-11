---
title: "cache和buffer"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"计算机科学"
]
tags : [
"CS",
"System"
]
series : [
"Manual"
]
images : [

]
---

[comment]: <> "# cache和buffer"

首先我们使用 `free -m` 查看系统内存的使用情况：

<img src="https://cdn.tkaid.com/img/20200502221926518.png" alt="老版本linux内核" style="zoom:50%;" />

可以看出，系统内存为 16G，Swap 内存 16G，mem free 虽然显示为 1118，因缓存的存在，不能认为系统目前内剩下这么多内存。而应该把 buffers、cached 的也算上，即 free+cached+buffers=1118+7110+430＝8658，总内存再减去 8658＝7314，与 buffers/cache 行中对应 free 列的 7312 和 8659 基本一致。

> 这里顺便介绍一下Swap，了解Swap更多详情请移步 👉 [Swap交换分区概念](https://www.cnblogs.com/kerrycode/p/5246383.html)

> Linux内核为了提高读写效率与速度，会将文件在内存中进行缓存，这部分内存就是Cache Memory(缓存内存)。即使你的程序运行结束后，Cache Memory也不会自动释放。这就会导致你在Linux系统中程序频繁读写文件后，你会发现可用物理内存变少。当系统的物理内存不够用的时候，就需要将物理内存中的一部分空间释放出来，以供当前运行的程序使用。那些被释放的空间可能来自一些很长时间没有什么操作的程序，这些被释放的空间被临时保存到Swap空间中，等到那些程序要运行时，再从Swap分区中恢复保存的数据到内存中。这样，系统总是在物理内存不够时，才进行Swap交换。

从Linux kernel version 2.4开始，buffer/cache合并（详情见 👉 [Linux IO的buffer cache和page cache合并的原因](https://blog.csdn.net/jasonchen_gbd/article/details/80151328)），所以同样使用 `free -m` 看到的是：

```sh
[root@k8s-node3 ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           7820        6577         147          30        1095         907
Swap:             0           0           0
```

> 使用 `hostnamectl` 或者 `cat /proc/version` 可以查看内核版本：

> ```sh
> [root@k8s-node3 ~]# hostnamectl
> Static hostname: k8s-node3
> Pretty hostname: VM-0-4-centos
>       Icon name: computer-vm
>         Chassis: vm
>      Machine ID: 8a248d59684442b3a2edd315013b7d1f
>         Boot ID: c14dd5b3b84646a58a98db62d5379832
>  Virtualization: kvm
> Operating System: CentOS Linux 7 (Core)
>     CPE OS Name: cpe:/o:centos:centos:7
>          Kernel: Linux 3.10.0-1127.19.1.el7.x86_64
>    Architecture: x86-64
> [root@k8s-node3 ~]# cat /proc/version
> Linux version 3.10.0-1127.19.1.el7.x86_64 (mockbuild@kbuilder.bsys.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) ) #1 SMP Tue Aug 25 17:23:54 UTC 2020
> ```

我们假设buffer/cache还没发生合并，那它们之间的区别是什么呢？

这里罗列几个高赞👍知乎回答：

[Cache 和 Buffer 都是缓存，主要区别是什么？](https://www.zhihu.com/question/26190832)

> - cache 是为了弥补高速设备和低速设备的鸿沟而引入的中间层，最终起到**加快访问速度**的作用。
> - 而 buffer 的主要目的进行流量整形，把突发的大数量较小规模的 I/O 整理成平稳的小数量较大规模的 I/O，以**减少响应次数**（比如从网上下电影，你不能下一点点数据就写一下硬盘，而是积攒一定量的数据以后一整块一起写，不然硬盘都要被你玩坏了）。

[Linux系统中的Page cache和Buffer cache](https://zhuanlan.zhihu.com/p/35277219)

> 磁盘的操作有逻辑级（文件系统）和物理级（磁盘块），这两种Cache就是分别缓存逻辑和物理级数据的。
>
> 假设我们通过文件系统操作文件，那么文件将被缓存到Page Cache，如果需要刷新文件的时候，Page Cache将交给Buffer Cache去完成，因为Buffer Cache就是缓存磁盘块的。
>
> 也就是说，直接去操作文件，那就是Page Cache区缓存，用dd等命令直接操作磁盘块，就是Buffer Cache缓存的东西。
>
> Page cache实际上是针对文件系统的，是文件的缓存，在文件层面上的数据会缓存到page cache。文件的逻辑层需要映射到实际的物理磁盘，这种映射关系由文件系统来完成。当page cache的数据需要刷新时，page cache中的数据交给buffer cache，但是这种处理在2.6版本的内核之后就变的很简单了，没有真正意义上的cache操作。
>
> Buffer cache是针对磁盘块的缓存，也就是在没有文件系统的情况下，直接对磁盘进行操作的数据会缓存到buffer cache中，例如，文件系统的元数据都会缓存到buffer cache中。
>
> 简单说来，page cache用来缓存文件数据，buffer cache用来缓存磁盘数据。在有文件系统的情况下，对文件操作，那么数据会缓存到page cache，如果直接采用dd等工具对磁盘进行读写，那么数据会缓存到buffer cache。
>
> **Buffer(Buffer Cache)以块形式缓冲了块设备的操作，定时或手动的同步到硬盘，它是为了缓冲写操作然后一次性将很多改动写入硬盘，避免频繁写硬盘，提高写入效率。**
>
> **Cache(Page Cache)以页面形式缓存了文件系统的文件，给需要使用的程序读取，它是为了给读操作提供缓冲，避免频繁读硬盘，提高读取效率。**



## 参考

[Linux 操作系统原理 — 内存 — Cache 和 Buffer](https://blog.csdn.net/Jmilk/article/details/105896326)

[Linux系统中的Page cache和Buffer cache](https://zhuanlan.zhihu.com/p/35277219)

[Cache 和 Buffer 都是缓存，主要区别是什么？](https://www.zhihu.com/question/26190832)

[Swap交换分区概念](
