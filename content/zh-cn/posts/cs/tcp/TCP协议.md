---
title: "TCP协议"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"计算机科学"
]
tags : [
"CS","TCP"
]
series : [
"Manual"
]
images : [

]
---

[comment]: <> "# TCP协议"

### 简介

数据在TCP层称为流（Stream），数据分组称为分段（Segment）。作为比较，数据在IP层称为Datagram，数据分组称为分片（Fragment）。 UDP 中分组称为Message。

### TCP 数据包的大小

以太网数据包（packet）的大小是固定的，最初是1518字节，后来增加到1522字节。其中， 1500 字节是负载（payload），22字节是头信息（head）。 IP 数据包在以太网数据包的负载里面，它也有自己的头信息，最少需要20字节，所以 IP 数据包的负载最多为1480字节。

<img src="https://cdn.tkaid.com/img/bg2012052913.png" alt="img" style="zoom:50%;" />

（图片说明：IP 数据包在以太网数据包里面，TCP 数据包在 IP 数据包里面。)

TCP 数据包在 IP 数据包的负载里面。它的头信息最少也需要20字节，因此 TCP 数据包的最大负载是 1480 - 20 = 1460 字节。由于 IP 和 TCP 协议往往有额外的头信息，所以 TCP 负载实际为1400字节左右。

因此，一条1500字节的信息需要两个 TCP 数据包。HTTP/2 协议的一大改进， 就是压缩 HTTP 协议的头信息，使得一个 HTTP 请求可以放在一个 TCP 数据包里面，而不是分成多个，这样就提高了速度。

### 创建连接

TCP用三次握手（或称三路握手，three-way handshake）过程创建一个连接。在连接创建过程中，很多参数要被初始化，例如序号被初始化以保证按序传输和连接的强壮性。 一对终端同时初始化一个它们之间的连接是可能的。但通常是由一端打开一个[套接字](https://zh.wikipedia.org/wiki/Berkeley套接字)（[socket](https://zh.wikipedia.org/wiki/Socket)）然后监听来自另一方的连接，这就是通常所指的被动打开（passive open）。服务器端被被动打开以后，用户端就能开始创建主动打开（active open）。

1. 客户端通过向服务器端发送一个SYN来创建一个主动打开，作为三次握手的一部分。客户端把这段连接的序号设定为随机数A。
2. 服务器端应当为一个合法的SYN回送一个SYN/ACK。ACK的确认码应为A+1，SYN/ACK包本身又有一个随机产生的序号B。
3. 最后，客户端再发送一个ACK。此时包的序号被设定为A+1，而ACK的确认码则为B+1。当服务端收到这个ACK的时候，就完成了三次握手，并进入了连接创建状态。

<img src="https://cdn.tkaid.com/img/Connection_TCP.png" alt="img" style="zoom: 50%;" />

注意：三次握手建立连接的时候， SYN/ACK 是一个数据包发送出去的

<img src="https://cdn.tkaid.com/img/%E4%B8%89%E6%AC%A1%E6%8F%A1%E6%89%8B.png" alt="img" style="zoom:60%;" />

注意：三次握手建立连接的时候， SYN/ACK 是一个数据包发送出去的

如果服务器端接到了客户端发的SYN后回了SYN-ACK后客户端掉线了，服务器端没有收到客户端回来的ACK，那么，这个连接处于一个中间状态，即没成功，也没失败。于是，服务器端如果在一定时间内没有收到的TCP会重发SYN-ACK。在Linux下，默认重试次数为5次，重试的间隔时间从1s开始每次都翻倍，5次的重试时间间隔为1s, 2s, 4s, 8s, 16s，总共31s，第5次发出后还要等32s才知道第5次也超时了，所以，总共需要 1s + 2s + 4s+ 8s+ 16s + 32s = 63s，TCP才会断开这个连接。使用三个TCP参数来调整行为：tcp_synack_retries 减少重试次数；tcp_max_syn_backlog，增大SYN连接数；tcp_abort_on_overflow决定超出能力时的行为。

### 数据传输

1. 发送方首先发送第一个包含序列号为1（可变化）和1460字节数据的TCP报文段给接收方。接收方以一个没有数据的TCP报文段来回复（只含报头），用确认号1461来表示已完全收到并请求下一个报文段。
2. 发送方然后发送第二个包含序列号为1461，长度为1460字节的数据的TCP报文段给接收方。正常情况下，接收方以一个没有数据的TCP报文段来回复，用确认号2921（1461+1460）来表示已完全收到并请求下一个报文段。发送接收这样继续下去。
3. 然而当这些数据包都是相连的情况下，接收方没有必要每一次都回应。比如，他收到第1到5条TCP报文段，只需回应第五条就行了。在例子中第3条TCP报文段被丢失了，所以尽管他收到了第4和5条，然而他只能回应第2条。
4. 发送方在发送了第三条以后，没能收到回应，因此当时钟（timer）过时（expire）时，他重发第三条。（每次发送者发送一条TCP报文段后，都会再次启动一次时钟：RTT）。
5. 这次第三条被成功接收，接收方可以直接确认第5条，因为4，5两条已收到。

<img src="https://cdn.tkaid.com/img/Tcp_transport_example.gif" alt="img" style="zoom: 67%;" />

上图只考虑了一方发送数据一方接收数据的情形，如果是双方都收发数据的情形则如下图所示：

<img src="https://cdn.tkaid.com/img/transport.jpg" alt="img" style="zoom:67%;" />

上图一共4次通信。第一次通信，A 主机发给B 主机的数据包编号是1，长度是100字节，因此第二次通信 B 主机的 ACK 编号是 1 + 100 = 101，第三次通信 A 主机的数据包编号也是 101。同理，第二次通信 B 主机发给 A 主机的数据包编号是1，长度是200字节，因此第三次通信 A 主机的 ACK 是201，第四次通信 B 主机的数据包编号也是201。

### 断开连接

所谓四次挥手（Four-Way Wavehand）即终止TCP连接，就是指断开一个TCP连接时，需要客户端和服务端总共发送4个包以确认连接的断开。在socket编程中，这一过程由客户端或服务端任一方执行close来触发，整个流程如下图所示：

<img src="https://cdn.tkaid.com/img/%E5%9B%9B%E6%AC%A1%E5%88%86%E6%89%8B.png" alt="img" style="zoom: 67%;" />

注意：四次分手断开连接的时候，ACK和FIN信号是分成两次发送的

由于TCP连接是全双工的，因此，每个方向都必须要单独进行关闭。这一原则是当一方完成数据发送任务后，发送一个FIN来终止这一方向的连接，收到一个FIN只是意味着这一方向上没有数据流动了，即不会再收到数据了，但是在这个TCP连接上仍然能够发送数据，直到这一方向也发送了FIN。首先进行关闭的一方将执行主动关闭，而另一方则执行被动关闭。

1. 第一次挥手：Client发送一个FIN，用来关闭Client到Server的数据传送，Client进入FIN_WAIT_1状态。
2. 第二次挥手：Server收到FIN后，发送一个ACK给Client，确认序号为收到序号+1（与SYN相同，一个FIN占用一个序号），Server进入CLOSE_WAIT状态。
3. 第三次挥手：Server发送一个FIN，用来关闭Server到Client的数据传送，Server进入LAST_ACK状态。
4. 第四次挥手：Client收到FIN后，Client进入TIME_WAIT状态，接着发送一个ACK给Server，确认序号为收到序号+1，Server进入CLOSED状态，完成四次挥手。 主动关闭端接收到FIN后，就发送ACK包，等待足够时间以确保被动关闭端收到了终止请求的确认包。【按照RFC 793，一个连接可以在TIME-WAIT保证最大四分钟，即[最大分段寿命](https://zh.wikipedia.org/wiki/最大分段寿命)（maximum segment lifetime）的2倍】

> 为什么建立连接是3次分手是4次？
> 首先，因为tcp是全双工通信，即双方都需要收发数据，因此双方都要通过发送syn指令来随机初始化一个包序列seq，因为ack和syn的包不大，被动打开的一方一次性发送了ack和syn指令给主动打开的一方，主动打开方再回复一个ack，所以总共3次。
> 而分手的时候，存在类单双工的情形（即只有一方发数据，另外一方收数据），所以不能在回复fin的时候同时发送fin，即需要分开发送ack和fin，所以一共4次通信。

#### 资源使用

主机收到一个TCP包时，用两端的IP地址与端口号来标识这个TCP包属于哪个session。使用一张表来存储所有的session，表中的每条称作Transmission Control Block（TCB），tcb结构的定义包括连接使用的源端口、目的端口、目的ip、序号、应答序号、对方窗口大小、己方窗口大小、tcp状态、tcp输入/输出队列、应用层输出队列、tcp的重传有关变量等。

服务器端的连接数量是无限的，只受内存的限制。客户端的连接数量，过去由于在发送第一个SYN到服务器之前需要先分配一个随机空闲的端口，这限制了客户端IP地址的对外发出连接的数量上限。从Linux 4.2开始，有了socket选项IP_BIND_ADDRESS_NO_PORT，它通知Linux内核不保留usingbind使用端口号为0时内部使用的临时端口（ephemeral port），在connect时会自动选择端口以组成独一无二的四元组（同一个客户端端口可用于连接不同的服务器[套接字](https://zh.wikipedia.org/wiki/套接字)；同一个服务器端口可用于接受不同客户端套接字的连接）。对于不能确认的包、接收但还没读取的数据，都会占用操作系统的资源。

## 参考

[传输控制协议](https://zh.wikipedia.org/wiki/传输控制协议)

[TCP 协议简介](http://www.ruanyifeng.com/blog/2017/06/tcp-protocol.html)

[图解TCP传输过程（三次握手、数据传输、四次挥手）](https://www.polarxiong.com/archives/图解TCP传输过程-三次握手-数据传输-四次挥手.html)

[TCP协议详解](https://www.jianshu.com/p/ef892323e68f)
