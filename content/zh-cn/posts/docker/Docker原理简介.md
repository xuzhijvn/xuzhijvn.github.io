+++
title = "Docker原理简介"
date = 2022-03-01T14:10:42+08:00
featured = false
comment = true
toc = true
reward = true
pinned = false
categories = [
  "容器"
]
tags = [
  "Docker"
]
series = [
  ""
]
images = []

+++

利用命名空间来做权限的隔离控制，利用cgroups来做资源分配

<!--more-->

Docker 底层的核心技术包括 Linux 上的 <font color=green>命名空间（Namespaces）</font>、<font color=green>控制组（Control groups）</font>、<font color=green>Union 文件系统（Union file systems）</font>和 <font color=green>容器格式（Container format）</font>。

我们知道，传统的虚拟机通过在宿主主机中运行 hypervisor 来模拟一整套完整的硬件环境提供给虚拟机的操作系统。虚拟机系统看到的环境是可限制的，也是彼此隔离的。 这种直接的做法实现了对资源最完整的封装，但很多时候往往意味着系统资源的浪费。 例如，以宿主机和虚拟机系统都为 Linux 系统为例，虚拟机中运行的应用其实可以利用宿主机系统中的运行环境。

假设你现在变身了，站在了 Docker 和 虚拟机的内部，从里面向外看，发现虚拟机有自己的 CPU(虚拟CPU)、内存、硬盘，再往外才是宿主机的 CPU、硬盘、内存等。而如果是在Docker内部向外看，发现你无论站在当前实体机的哪个容器里，看到的都是宿主机的 CPU、硬盘、内存等。说明 Dokcer 容器是直接拿宿主机的资源当自己的用，所以每个容器的硬件配置都是一样的，而虚拟机是完全虚拟出来一套。

我们知道，在操作系统中，包括内核、文件系统、网络、PID、UID、IPC、内存、硬盘、CPU 等等，所有的资源都是应用进程直接共享的。 要想实现虚拟化，除了要实现对内存、CPU、网络IO、硬盘IO、存储空间等的限制外，还要实现文件系统、网络、PID、UID、IPC等等的相互隔离。 前者相对容易实现一些，后者则需要宿主机系统的深入支持。

随着 Linux 系统对于命名空间功能的完善实现，程序员已经可以实现上面的所有需求，让某些进程在彼此隔离的命名空间中运行。大家虽然都共用一个内核和某些运行时环境（例如一些系统命令和系统库），但是彼此却看不到，都以为系统中只有自己的存在。这种机制就是容器（Container），利用<font color=red>命名空间</font>来做权限的隔离控制，利用 <font color=red>cgroups </font>来做资源分配。



## 参考

[底层实现](https://yeasy.gitbook.io/docker_practice/underly)
[用 Docker 瞬间搭建本地开发环境](https://www.51cto.com/article/761653.html)
