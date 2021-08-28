---
title: "5种IO模型"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"计算机科学"
]
tags : [
"IO",
"CS"
]
series : [
"Manual"
]
images : [
"images/center.png"
]
---


[comment]: <> (# 5种IO模型)

## 1. 前言

W. Richard Stevens在 《Unix Network Programming Volume 1 3rd Edition - The Sockets Networking API》文中的6.2 I/O Models小节中对如下5中IO模型进行了详细的阐述：

- `blocking I/O`
- `nonblocking I/O`
- `I/O multiplexing (select and poll)`
- `signal driven I/O (SIGIO)`
- `asynchronous I/O (the POSIX aio_functions)`

由signal driven IO在实际中并不常用，所以主要介绍其余四种IO Model。

再说一下IO发生时涉及的对象和步骤。对于一个network IO (这里我们以read举例)，它会涉及到两个系统对象，一个是调用这个IO的process (or thread)，另一个就是系统内核(kernel)。当一个read操作发生时，它会经历两个阶段：

- `等待数据准备` (Waiting for the data to be ready)（将数据从网卡读到内核）
- `将数据从内核拷贝到进程中`(Copying the data from the kernel to the process)

记住这两点很重要，因为这些IO模型的区别就是在两个阶段上各有不同的情况。

## 2. 概念说明

在进行解释之前，首先要说明几个概念：

- `用户空间和内核空间`
- `进程切换`
- `进程的阻塞`
- `文件描述符`
- `缓存 I/O`

### 2.1 用户空间与内核空间

现在操作系统都是采用虚拟存储器，那么对32位操作系统而言，它的寻址空间（虚拟存储空间）为4G（2的32次方）。操作系统的核心是内核，独立于普通的应用程序，可以访问受保护的内存空间，也有访问底层硬件设备的所有权限。为了保证用户进程不能直接操作内核（kernel），保证内核的安全，操心系统将虚拟空间划分为两部分，一部分为内核空间，一部分为用户空间。针对linux操作系统而言，将最高的1G字节（从虚拟地址`0xC0000000`到`0xFFFFFFFF`），供内核使用，称为内核空间，而将较低的3G字节（从虚拟地址`0x00000000`到`0xBFFFFFFF`），供各个进程使用，称为用户空间。

### 2.2 进程切换

为了控制进程的执行，内核必须有能力挂起正在CPU上运行的进程，并恢复以前挂起的某个进程的执行。这种行为被称为进程切换。因此可以说，任何进程都是在操作系统内核的支持下运行的，是与内核紧密相关的。
从一个进程的运行转到另一个进程上运行，这个过程中经过下面这些变化：

1. 保存处理机上下文，包括程序计数器和其他寄存器。

2. 更新PCB信息。

3. 把进程的PCB移入相应的队列，如就绪、在某事件阻塞等队列。

4. 选择另一个进程执行，并更新其PCB。

5. 更新内存管理的数据结构。

6. 恢复处理机上下文。

   注：总而言之就是很耗资源

### 2.3 进程的阻塞

正在执行的进程，由于期待的某些事件未发生，如请求系统资源失败、等待某种操作的完成、新数据尚未到达或无新工作做等，则由系统自动执行阻塞原语(Block)，使自己由运行状态变为阻塞状态。可见，进程的阻塞是进程自身的一种主动行为，也因此只有处于运行态的进程（获得CPU），才可能将其转为阻塞状态。当进程进入阻塞状态，是不占用CPU资源的。

### 2.4 文件描述符fd

`文件描述符（File descriptor）`是计算机科学中的一个术语，是一个用于表述指向文件的引用的抽象化概念。
文件描述符在形式上是一个非负整数。实际上，它是一个索引值，指向内核为每一个进程所维护的该进程打开文件的记录表。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。在程序设计中，一些涉及底层的程序编写往往会围绕着文件描述符展开。但是文件描述符这一概念往往只适用于UNIX、Linux这样的操作系统。

### 2.5 缓存 I/O

`缓存 I/O` 又被称作`标准 I/O`，大多数文件系统的默认 I/O 操作都是缓存 I/O。在 Linux 的缓存 I/O 机制中，操作系统会将 I/O 的数据缓存在文件系统的页缓存（ page cache ）中，也就是说，数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间。

> 缓存 I/O 的缺点：
> 数据在传输过程中需要在应用程序地址空间和内核进行多次数据拷贝操作，这些数据拷贝操作所带来的 CPU 以及内存开销是



## 3. IO模型

### 3.1 blocking IO

在linux中，默认情况下所有的socket都是blocking，一个典型的读操作流程大概是这样：

![img](https://picgo.6and.ltd/img/img_5f3e145553aec-1024x559-20210621141130881.png)
当用户进程调用了`recvfrom`这个系统调用，kernel就开始了IO的第一个阶段：准备数据（对于网络IO来说，很多时候数据在一开始还没有到达。比如，还没有收到一个完整的UDP包。这个时候kernel就要等待足够的数据到来）。这个过程需要等待，也就是说数据被拷贝到操作系统内核的缓冲区中是需要一个过程的。而在用户进程这边，整个进程会被阻塞（当然，是进程自己选择的阻塞)。当kernel一直等到数据准备好了，它就会将数据从kernel中拷贝到用户内存，然后kernel返回结果，用户进程才解除block的状态，重新运行起来。

> 所以，blocking IO的特点就是在IO执行的两个阶段都被block了。
>

### 3.2 nonblocking IO

linux下，可以通过设置socket使其变为non-blocking。当对一个non-blocking socket执行读操作时，流程是这个样子：

![img](https://picgo.6and.ltd/img/img_5f3e143f07ef6-1024x559-20210621141137000.png)
当用户进程发出read操作时，如果kernel中的数据还没有准备好，那么它并不会block用户进程，而是立刻返回一个`EWOULDBLOCK`的error。从用户进程角度讲 ，它发起一个read操作后，并不需要等待，而是马上就得到了一个结果。用户进程判断结果是一个error时，它就知道数据还没有准备好，于是它可以再次发送read操作。一旦kernel中的数据准备好了，并且又再次收到了用户进程的`system call`，那么它马上就将数据拷贝到了用户内存，然后返回。

> 所以，nonblocking IO的特点是用户进程需要不断的主动询问kernel数据好了没有。
>

### 3.3 IO multiplexing

IO multiplexing就是我们说的`select`，`poll`，`epoll`，有些地方也称这种IO方式为`event driven IO`。select/epoll的好处就在于单个process就可以同时处理多个网络连接的IO。它的基本原理就是select，poll，epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。

![img](https://picgo.6and.ltd/img/img_5f3e14163bf52-1024x534-20210621141144263.png)

当用户进程调用了select，那么整个进程会被block，而同时，kernel会“监视”所有select负责的socket，当任何一个socket中的数据准备好了，select就会返回。这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程。

这个图和blocking IO的图其实并没有太大的不同，事实上，还更差一些。因为这里需要使用两个system call (`select`和 `recvfrom`)，而blocking IO只调用了一个system call (`recvfrom`)。但是，用select的优势在于它可以同时处理多个connection。

所以，如果处理的连接数不是很高的话，使用select/epoll的web server不一定比使用multi-threading + blocking IO的web server性能更好，可能延迟还更大。select/epoll的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接。

在I/O编程过程中，当需要同时处理多个客户端接入请求时，可以利用多线程或者I/O多路复用技术进行处理。I/O多路复用技术通过把多个I/O的阻塞复用到同一个select的阻塞上，从而使得系统在单线程的情况下可以同时处理多个客户端请求。与传统的多线程/多进程模型比，I/O多路复用的最大优势是系统开销小，系统不需要创建新的额外进程或者线程，也不需要维护这些进程和线程的运行，降底了系统的维护工作量，节省了系统资源。

> 所以，I/O 多路复用的特点是通过一种机制一个进程能同时等待多个文件描述符，而这些文件描述符（套接字描述符）其中的任意一个进入读就绪状态，select()函数就可以返回。
>

### 3.4 signal-driven IO

信号驱动式I/O：首先我们允许Socket进行信号驱动IO,并安装一个信号处理函数，进程继续运行并不阻塞。当数据准备好时，进程会收到一个SIGIO信号，可以在信号处理函数中调用I/O操作函数处理数据。过程如下图所示：

 

![img](https://picgo.6and.ltd/img/img_5f3def904eae9-1024x651.png)

### 3.5 asynchronous IO

相对于同步IO，异步IO不是顺序执行。用户进程进行`aio_read`系统调用之后，无论内核数据是否准备好，都会直接返回给用户进程，然后用户态进程可以去做别的事情。等到socket数据准备好了，内核直接复制数据给进程，然后从内核向进程发送通知。IO两个阶段，进程都是非阻塞的。

Linux提供了AIO库函数实现异步，但是用的很少。目前有很多开源的异步IO库，例如`libevent、libev、libuv`。异步过程如下图所示：

![img](https://picgo.6and.ltd/img/img_5f3e14a73cc23-1024x644-20210621141152724.png)

用户进程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从kernel的角度，当它受到一个asynchronous read之后，首先它会立刻返回，所以不会对用户进程产生任何block。然后，kernel会等待数据准备完成，然后将数据拷贝到用户内存，当这一切都完成之后，kernel会给用户进程发送一个signal，告诉它read操作完成了。

> 所以，aio的特点是两个阶段都不阻塞，特别是第二阶段，数据从内核空间拷贝到用户空间是是系统完成的，系统完成拷贝后再通知应用进程。
>

## 4. 总结

![img](https://picgo.6and.ltd/img/img_5f3df05eac237-1024x656-20210621141201925.png)

 

## 5. 参考资料

[W. Richard Stevens - Unix Network Programming Volume 1 3rd Edition - The Sockets Networking API.pdf](https://docs.google.com/viewer?a=v&pid=sites&srcid=ZGVmYXVsdGRvbWFpbnxyYWdodXNpdGNzZXxneDo2NzU3YWQ4NDNmZTU5M2Yz)

[聊聊Linux 五种IO模型 猿码架构](https://www.jianshu.com/p/486b0965c296)

[Linux IO模式及 select、poll、epoll详解](https://segmentfault.com/a/1190000003063859)
