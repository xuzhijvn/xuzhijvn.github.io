---
title: "深度图解Redis Cluster原理"
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

]
---

[comment]: <> "# 深度图解Redis Cluster原理"

主从能实现**读能力**进行扩展，但无法**自动故障切换**、**写能力**和**存储能力**；

哨兵能**自动故障切换**，但无法对**写能力**和**存储能力**是无法进行扩展；

集群能**读能力**、**写能力**、**存储能力**进行扩展，也能**自动故障切换**

## 前言

上文我们聊了基于Sentinel的Redis高可用架构，了解了Redis基于读写分离的主从架构，同时也知道当Redis的master发生故障之后，Sentinel集群是如何执行failover的，以及其执行failover的原理是什么。

这里大概再提一下，Sentinel集群会对Redis的主从架构中的Redis实例进行监控，一旦发现了master节点宕机了，就会选举出一个Sentinel节点来执行故障转移，从原来的slave节点中选举出一个，将其提升为master节点，然后让其他的节点去复制新选举出来的master节点。

你可能会觉得这样没有问题啊，甚至能够满足我们生产环境的使用需求了，那我们为什么还需要Redis Cluster呢？

## 1. 为什么需要Redis Cluster

的确，在**数据**上，有replication副本做保证；**可用性**上，master宕机会自动的执行failover。

> 那问题在哪儿呢？

首先Redis Sentinel说白了也是基于**主从复制**，在主从复制中slave的数据是完全来自于master。

<img src="https://cdn.tkaid.com/img/0081Kckwly1gll2khn7srj30el03vt8o.jpg" alt="redis-capacity" style="zoom:67%;" />

假设master节点的内存只有4G，那slave节点所能存储的数据上限也只能是4G。而且在之前的[跟随杠精的视角一起来了解Redis的主从复制](https://mp.weixin.qq.com/s/VJTBmAB-A1aRT9DR6v5gow)文章中也说过，主从复制架构中是读写分离的，我们可以通过增加slave节点来扩展主从的读并发能力，但是**写能力**和**存储能力**是无法进行扩展的，就只能是master节点能够承载的上限。

所以，当你只需要存储4G的数据时候的，基于主从复制和基于Sentinel的高可用架构是完全够用的。

但是如果当你面临的是海量的数据的时候呢？16G、64G、256G甚至1T呢？现在互联网的业务里面，如果你的体量足够大，我觉得是肯定会面临缓存**海量**缓存数据的场景的。

这就是为什么我们需要引入**Redis Cluster**。

## 2. Redis Cluster是什么

知道了为什么需要Redis Cluster之后，我们就可以来对其一探究竟了。

> 那什么是Redis Cluster呢？

很简单，你就可以理解为n个主从架构组合在一起对外服务。Redis Cluster要求至少需要3个master才能组成一个集群，同时每个master至少需要有一个slave节点。

<img src="https://cdn.tkaid.com/img/0081Kckwly1gll36kwg27j30qr0i3myk.jpg" alt="redis-cluster" style="zoom: 67%;" />

这样一来，如果一个主从能够存储32G的数据，如果这个集群包含了两个主从，则整个集群就能够存储64G的数据。

我们知道，主从架构中，可以通过增加slave节点的方式来扩展读请求的并发量，那Redis Cluster中是如何做的呢？虽然每个master下都挂载了一个slave节点，但是在Redis Cluster中的读、写请求其实都是在**master**上完成的。

slave节点只是充当了一个数据备份的角色，当master发生了宕机，就会将对应的slave节点提拔为master，来重新对外提供服务。

## 3. 节点负载均衡

知道了什么是Redis Cluster，我们就可以继续下面的讨论了。

不知道你思考过一个问题没，这么多的master节点。我存储的时候，到底该选择哪个节点呢？一般这种负载均衡算法，会选择**哈希算法**。哈希算法是怎么做的呢？

<img src="https://cdn.tkaid.com/img/0081Kckwly1gll3oxx5iqj30d80dhaag.jpg" alt="redis-master-select" style="zoom:67%;" />

首先就是对key计算出一个hash值，然后用哈希值对master数量进行取模。由此就可以将key负载均衡到每一个Redis节点上去。这就是简单的**哈希算法**的实现。

那Redis Cluster是采取的上面的哈希算法吗？答案是**没有**。

Redis Cluster其实采取的是类似于**一致性哈希**的算法来实现节点选择的。那为什么不用哈希算法来进行实例选择呢？以及为什么说是类似的呢？我们继续讨论。

因为如果此时某一台master发生了宕机，那么此时会导致Redis中**所有的缓存失效**。为什么是所有的？假设之前有3个master，那么之前的算法应该是 hash % 3，但是如果其中一台master宕机了，则算法就会变成 hash % 2，会影响到之前存储的所有的key。而这对缓存后面保护的DB来说，是致命的打击。

## 4. 什么是一致性哈希

知道了通过传统哈希算法来实现对节点的负载均衡的弊端，我们就需要进一步了解**什么是一致性哈希**。

我们上面提过哈希算法是对master实例数量来取模，而**一致性哈希**则是对2^32取模，也就是值的范围在[0, 2^32 -1]。一致性哈希将其范围抽象成了一个圆环，使用CRC16算法计算出来的哈希值会落到圆环上的某个地方。

然后我们的Redis实例也分布在圆环上，我们在圆环上按照顺时针的顺序找到第一个Redis实例，这样就完成了对key的节点分配。我们举个例子。

<img src="https://cdn.tkaid.com/img/0081Kckwgy1glmcvvmyvkj30ee0jdmy3.jpg" alt="hash" style="zoom: 50%;" />

假设我们有A、B、C三个Redis实例按照如图所示的位置分布在圆环上，此时计算出来的hash值，取模之后位置落在了**位置D**，那么我们按照顺时针的顺序，就能够找到我们这个key应该分配的Redis实例B。同理如果我们计算出来位置在E，那么对应选择的Redis的实例就是A。

即使这个时候Redis实例B挂了，也不会影响到实例A和C的缓存。

<img src="https://cdn.tkaid.com/img/0081Kckwgy1gln53ifhuwj30ee0jdt9p.jpg" alt="hash-down" style="zoom: 50%;" />

例如此时节点B挂了，那之前计算出来在位置D的key，此时会按照顺时针的顺序，找到节点C。相当于自动的把原来节点B的流量给转移到了节点C上去。而其他原本就在节点A和节点C的数据则完全不受影响。

这就是一致性哈希，能够在我们后续需要新增节点或者删除节点的时候，不影响其他节点的正常运行。

## 5. 虚拟节点机制

但是一致性哈希也存在自身的小问题，例如当我们的Redis节点分布如下时，就有问题了。

<img src="https://cdn.tkaid.com/img/0081Kckwgy1gln5gdf877j30ee0jd0to.jpg" alt="hash-unevently" style="zoom: 50%;" />

此时数据落在节点A上的概率明显是大于其他两个节点的，其次落在节点C上的概率最小。这样一来会导致整个集群的数据存储不平衡，AB节点压力较大，而C节点资源利用不充分。为了解决这个问题，一致性哈希算法引入了**虚拟节点机制**。

<img src="https://cdn.tkaid.com/img/0081Kckwgy1gln5lb847oj30ee0k2my8.jpg" alt="virtual-dom" style="zoom: 50%;" />

在圆环中，增加了对应节点的虚拟节点，然后完成了虚拟节点到真实节点的映射。假设现在计算得出了位置D，那么按照顺时针的顺序，我们找到的第一个节点就是**C #1**，最终数据实际还是会落在节点C上。

通过增加虚拟节点的方式，使ABC三个节点在圆环上的位置更加均匀，平均了落在每一个节点上的概率。这样一来就解决了上文提到的数据存储存在不均匀的问题了，这就是一致性哈希的虚拟节点机制。

## 6. Redis Cluster采用的什么算法

上面提到过，Redis Cluster采用的是类一致性哈希算法，之所以是**类一致性哈希算法**是因为它们实现的方式还略微有差别。

例如一致性哈希是对2^32取模，而Redis Cluster则是对2^14（也就是16384）取模。Redis Cluster将自己分成了16384个**Slot**（槽位）。通过CRC16算法计算出来的哈希值会跟16384取模，取模之后得到的值就是对应的槽位，然后每个Redis节点都会负责处理一部分的槽位，就像下表这样。

| 节点 |   处理槽位    |
| :--: | :-----------: |
|  A   |   0 - 5000    |
|  B   | 5001 - 10000  |
|  C   | 10001 - 16383 |

每个Redis实例会自己维护一份**slot - Redis节点**的映射关系，假设你在节点A上设置了某个key，但是这个key通过CRC16计算出来的槽位是由节点B维护的，那么就会提示你需要去节点B上进行操作。

<img src="https://cdn.tkaid.com/img/0081Kckwgy1glnlpufb7yj30f50bmjry.jpg" alt="slot-to-node" style="zoom:67%;" />

## 7. Redis Cluster如何做到高可用

不知道你思考过一个问题没，如果Redis Cluster中的某个master节点挂了，它是如何保证集群自身的高可用的？如果这个时候我们集群需要扩容节点，它该负责哪些槽位呢？我们一个一个问题的来看一下。

### 集群如何进行扩容

我们开篇聊过，Redis Cluster可以很方便的进行横向扩容，那当新的节点加入进来的时候，它是如何获取对应的slot的呢？

答案是通过**reshard**（重新分片）来实现。reshard可以将已经分配给某个节点的任意数量的slot迁移给另一个节点，在Redis内部是由redis-trib负责执行的。你可以理解为Redis其实已经封装好了所有的命令，而redis-trib则负责向获取slot的节点和被转移slot的节点发送命令来最终实现reshard。

假设我们需要向集群中加入一个D节点，而此时集群内已经有A、B、C三个节点了。

此时redis-trib会向A、B、C三个节点发送迁移出槽位的请求，同时向D节点发送准备导入槽位的请求，做好准备之后A、B、C这三个源节点就开始执行迁移，将对应的slot所对应的键值对迁移至目标节点D。最后redis-trib会向集群中所有主节点发送槽位的变更信息。

### 高可用及故障转移

Redis Cluster中保证集群高可用的思路和实现和Redis Sentinel如出一辙，感兴趣的可以去看我之前写的关于Sentinel的文章[Redis Sentinel-深入浅出原理和实战](https://mp.weixin.qq.com/s/k-wGpBBnS53Ap86KNiBYvA)。

简单来说，针对A节点，某一个节点认为A宕机了，那么此时是**主观宕机**。而如果集群内超过半数的节点认为A挂了， 那么此时A就会被标记为**客观宕机**。

一旦节点A被标记为了客观宕机，集群就会开始执行**故障转移**。其余正常运行的master节点会进行投票选举，从A节点的slave节点中选举出一个，将其切换成新的master对外提供服务。当某个slave获得了超过半数的master节点投票，就成功当选。

<img src="https://cdn.tkaid.com/img/0081Kckwgy1glqmfywf8jj30920f3mxp.jpg" alt="cluster-failover" style="zoom:67%;" />

当选成功之后，新的master会执行`slaveof no one`来让自己停止复制A节点，使自己成为master。然后将A节点所负责处理的slot，全部转移给自己，然后就会向集群发**PONG**消息来广播自己的最新状态。

按照**一致性哈希**的思想，如果某个节点挂了，那么就会沿着那个圆环，按照顺时针的顺序找到遇到的第一个Redis实例。

而对于Redis Cluster，某个key它其实并不关心它最终要去到哪个节点，他只关心他最终落到哪个slot上，无论你节点怎么去迁移，最终还是只需要找到对应的slot，然后再找到slot关联的节点，最终就能够找到最终的Redis实例了。

那这个**PONG**消息又是什么东西呢？别急，下面就会聊到。

## 8. 简单了解gossip协议

这就是Redis Cluster各个节点之间交换数据、通信所采用的一种协议，叫做**gossip**。

```
gossip: 流言、八卦、小道消息
```

gossip是在1989年的论文上提出的，我看了一堆资料都说的是1987年发表的，但是文章里的时间明确是1989年1月份发表。

<img src="https://cdn.tkaid.com/img/0081Kckwgy1globdud52sj30p0044dfx.jpg" alt="image-20201215100703648" style="zoom:67%;" />

感兴趣的可以去看看[Epidemic Algorithms for Replicated . Database Maintenance](http://bitsavers.trailing-edge.com/pdf/xerox/parc/techReports/CSL-89-1_Epidemic_Algorithms_for_Replicated_Database_Maintenance.pdf)，在当时提出gossip主要是为了解决在分布式数据库中，各个副本节点的数据同步问题。但随着技术的发展，gossip后续也被广泛运用于信息扩散、故障探测等等。

Redis Cluster就是利用了gossip来实现自身的**信息扩散**的。那使用gossip具体是如何通信的呢？

<img src="https://cdn.tkaid.com/img/0081Kckwgy1glob136uztj30hn0cwgmi.jpg" alt="gossip" style="zoom:67%;" />

很简单，就像图里这样。每个Redis节点**每秒钟**都会向其他的节点发送**PING**，然后被**PING**的节点会回一个**PONG**。

## 9. gossip协议消息类型

Redis Cluster中，节点之间的消息类型有5种，分别是MEET、PING、PONG、FAIL和PUBLISH。这些消息分别传递了什么内容呢？我简单总结了一下。

| 消息类型 | 消息内容                                                     |
| :------- | :----------------------------------------------------------- |
| MEET     | 给某个节点发送MEET消息，请求接收消息的节点加入到集群中       |
| PING     | 每隔一秒钟，选择5个最久没有通信的节点，发送PING消息，检测对应的节点是否在线；同时还有一种策略是，如果某个节点的通信延迟大于了`cluster-node-time`的值的一半，就会立即给该节点发送PING消息，避免数据交换延迟过久 |
| PONG     | 当节点接收到MEET或者PING消息之后，会回一个PONG消息给发送方，代表自己收到了MEET或者PING消息。同时，节点也可以主动的通过PONG消息向集群中广播自己的信息，让其他节点获取到自己最新的属性，就像完成了故障转移之后新的master向集群发送PONG消息一样 |
| FAIL     | 用于广播自己的对某个节点的宕机判断，假设当前节点对A节点判断为宕机，就会立即向Redis Cluster广播自己对于A节点的判断，所有收到消息的节点就会对A节点做标记 |
| PUBLISH  | 用于向指定的Channel发送消息，某个节点收到PUBLISH消息之后会直接在集群内广播，这样一来，客户端无论连接到任何节点都能够订阅这个Channel |

## 10. 使用gossip的优劣

既然Redis Cluster选择了gossip，那肯定存在一些gossip的优点，我们接下来简单梳理一下。

| 优点       | 描述                                                         |
| :--------- | :----------------------------------------------------------- |
| 扩展性     | 网络可以允许节点的任意增加和减少，新增加的节点的状态最终会与其他节点一致。 |
| 容错性     | 由于每个节点都持有一份完整元数据，所以任何节点宕机都不会影响gossip的运行 |
| 健壮性     | 与容错性类似，由于所有节点都持有数据，地位平台，是一个去中心化的设计，任何节点都不会影响到服务的运行 |
| 最终一致性 | 当有新的信息需要传递时，消息可以快速的发送到所有的节点，让所有的节点都拥有最新的数据 |

gossip可以在O(logN) 轮就可以将信息传播到所有的节点，为什么是O(logN)呢？因为每次ping，当前节点会带上自己的信息外加整个Cluster的1/10数量的节点信息，一起发送出去。你可以简单的把这个模型抽象为：

> 你转发了一个特别有意思的文章到朋友圈，然后你的朋友们都觉得还不错，于是就一传十、十传百这样的散播出去了，这就是朋友圈的**裂变传播**。

当然，gossip仍然存在一些缺点。例如消息可能最终会经过很多轮才能到达目标节点，而这可能会带来较大的延迟。同时由于节点会随机选出5个最久没有通信的节点，这可能会造成某一个节点同时收到n个重复的消息。

> **缺点**
>
> - **消息延迟** 节点随机向少数几个节点发送消息，消息最终是通过多个轮次的散播而到达全网，不可避免的造成消息延迟
> - **消息冗余** 节点定期随机选择周围节点发送消息，而收到消息的节点也会重复该步骤；不可避免的引起同一节点消息多次接收，增加消息处理压力，一般通过**文件校验和、缓存节点列表**等方式来进行优化减少数据对比带来的性能损耗
>
> 基于以上优缺点分析，`Gossip协议`满足**CAP**分布式理论中基于**AP**场景的数据最终一致性处理，常见应用有：`P2P网络通信、Apache Cassandra、Redis Cluster、Consul`等，还有`Apache Gossip`框架的开源实现供Gossip协议的学习。

## 11. 总结

总的来说，Redis Cluster相当于是把Redis的**主从架构**和**Sentinel**集成到了一起，从Redis Cluster的高可用机制、判断故障转移以及执行故障转移的过程，都和主从、Sentinel相关，这也是为什么我在之前的文章里说，主从是Redis高可用架构的基石。

## 参考

[深度图解Redis Cluster原理](https://www.cnblogs.com/detectiveHLH/p/14154665.html)

[[分布式系列]Gossip协议](https://juejin.cn/post/6930774718114955278#heading-17)
