+++
title = "Note"
description = ""
date = 2021-10-03T11:20:44+08:00
featured = false
draft = true
comment = true
toc = true
reward = true
categories = [
  ""
]
tags = [
  ""
]
series = []
images = []
+++

<!--more-->

创新实验室，半研究性质的，研究成果转化的团队。我负责项目开发，包括区块链平台，分布式共识算法优化，智能监控系统；先进技术预研，包括搭建卷积神经网络用于图像质量分析、信用卡风险识别。

风控事业部，业务开发，组内基础设施。

### 为什么跳槽？

**银行：**

企业氛围：受不了官本位那一套；受不了今天开展批评与自我批民主生活会，明天开展这讲话那精神专题学习。

公司对团队定位不明确，身心俱疲：又有版本任务，又要写各种材料专利、技术方案评审、联席方案评审、前瞻性研究报告、行业趋势分析报告

**货拉拉：**

团队缺乏技术氛围：居然没有定期的技术分享、深度也不够，找运维要中间件这没有那不支持的

岗位缺乏挑战：驾轻就熟，而且缺乏成长空间，这使得我很焦虑。



### 网关性能指标

**流量网关**：高性能，与具体的业务耦合少，专注转发，不用引入业务逻辑，不用维护lua脚本，不要专门的lua团队

**业务网关**：性能较高（没有指数级别的差距），与业务、技术栈相关，比如要通过Jar包和nacos、sential配合使用

Mock接口模拟20ms时延，报文大小约2K。

4核8G

| 分类     | 产品                          | 600并发 QPS | 600并发 90% Latency(ms) | 1000并发 QPS | 1000并发 90% Latency(ms) |
| -------- | ----------------------------- | ----------- | ----------------------- | ------------ | ------------------------ |
| 后端服务 | 直接访问后端服务              | 23540       | 32.19                   | 27325        | 52.09                    |
| 流量网关 | kong v2.4.1                   | 15662       | 50.87                   | 17152        | 84.3                     |
| 应用网关 | fizz-gateway-community v2.0.0 | 12206       | 65.76                   | 12766        | 100.34                   |
| 应用网关 | spring-cloud-gateway v2.2.9   | 11323       | 68.57                   | 10472        | 127.59                   |
| 应用网关 | shenyu v2.3.0                 | 9284        | 92.98                   | 9939         | 148.61                   |

### hashmap扩容机制

当map中包含的Entry的数量大于等于threshold = loadFactor * capacity的时候，且新建的Entry刚好落在一个非空的桶上，此刻触发扩容机制，将其容量扩大为2倍。调用resize扩容。

### ConcurrentHashMap

**JDK7**

数忿吉构：ReentrantLock + Segment + HashEntry, 是一个链表结构 元素查询二二次hash第一次Hash定位到Segment 

### 印象深刻的经历

一次缓存优化经历

Redis使用频率很高，流量高峰期千兆网卡跑满，导致数据读取非常慢。

**具体来说**：假设应用接口的总访问次数是1000万，总缓存数据量大小50KB，一天要承受 500G 的数据流，平均每秒钟是 5.78M 的数据，高峰期按50倍估算，也就是说高峰期 Redis 服务每秒要传输超过 250 MB的数据，而千兆网卡的带宽只在120多MB。

此时，立马能想到的解决办法是：1. 升级到万兆网卡。2. 使用redis cluster模式，并增加redis节点数量，将流量分摊多多台机器上

但是，云服务器升级万兆网卡困难。增加节点使得成本攀升，并且此时数据量并不大，服务器负载并不高，仅仅是redis服务器数据吞吐量大。

另辟蹊径：先访问本地内存，再访问redis，降低redis的访问频率（节点间数据同步的方案 —— Redis Pub/Sub ,  JGroups , MQ）。

其他类似方案的缺陷：**Ehcache集群模式** 在应用节点之间传递完整缓存数据，这样岂不是会把应用服务器的带宽跑满？而，

J2Cache的问题：如果应用发生重启，本地缓存超时时间丢失。

未来方向：封装成一个进程，服务器安装后，其他开发语言的应用可以通过客户端连接使用。



### go channel

https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/

### netty面试

### spring单例vs多例

Spring Bean有五种作用域：

`singleton` ,  `prototype`,  `request` , `session` ,  `global session`

默认是单例，可以通过xml配置bean多例，更方便的是用@Scope("prototype")注解。

Controller, Service, DAO默认都是单例的。

单例是线程不安全的，所以不要在类中使用状态可变的成员变量。

### @Autowired 和 @Resourse区别

两者都可以写在字段和setter方法上

@Autowired是Spring中定义注解，@Resourse是JRE定义的注解

@Autowired默认按byType装配，需要按byName来装配，可以结合@Qualifier注解一起使用，@Resourse默认按byName装配，它有可以配置的属性 name和type

```java
public class TestServiceImpl {
    @Autowired
    @Qualifier("userDao")
    private UserDao userDao; 
}
```



### tcp三握四分 + SpringMVC 打开网站看



### Spring设计模式

适配器 + 工厂 + 代理 + 单例

### 数据库深分页优化

成因：大量回表 👉🏿 大量随机IO

> 即使前10000个会扔掉，mysql也会通过二级索引上的主键id，去聚簇索引上查一遍数据，这可是10000次随机io，自然慢成哈士奇。 这里可能会提出疑问，为什么会有这种行为，这是mysql优化器的坑，至今也没解决。

**子查询优化：** 如下只有10次回表

```sql
select xxx,xxx from in (select id from table where second_index = xxx limit 10 offset 10000) 
```

**书签记录：**通过主键索引优化

```sql
SELECT * FROM cps_user_order_detail d WHERE d.id > #{maxId} AND d.order_time>'2020-8-5 00:00:00' ORDER BY d.order_time desc LIMIT 6;
```

id要是递增，不能跳页

**ES + scroll + search_after**

滚动翻页

### RabbitMQ交换机类型

<img src="https://pic2.zhimg.com/80/v2-a8594e4f7fec1495e692bdd1dc152d19_1440w.jpg" alt="img" style="zoom:67%;" />

Exchange 的三种主要类型：`Fanout`、`Direct` 和 `Topic`

Fanout Exchange 会忽略 RoutingKey 的设置，直接将 Message 广播到所有绑定的 Queue 中，`广播`

Direct Exchange 是 RabbitMQ 默认的 Exchange，对 RoutingKey 是精确匹配，`单播`

Topic Exchange 支持对 RoutingKey 模糊匹配，`多播`

### Java通过Executors提供四种线程池

分别为：

- **newCachedThreadPool**创建一个可缓存线程池，如果**线程池长度**超过处理需要，可灵活回收空闲线程，若无可用线程，则新建线程。
- **newFixedThreadPool**创建一个***\*定长线程池\****，可控制线程最大并发数，超出的线程会在队列中等待。
- ***\*newScheduledThreadPool\****创建一个**定长线程池**，支持定时及周期性任务执行。
- ***\*newSingleThreadExecutor\**** 创建一个**单线程化的线程池**，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。



### 常用Http状态码



301 Moved Permanently ：永久重定向（优先使用301）

302 Found（临时重定向）



401 Unauthorized

403 Forbidden：资源不可用，防盗链

429 Too Many Requests



500 Internal Server Error

502 Bad Gateway

503 Service Unavailable

504 Gateway Timeout

### 3个问题，技术栈是什么？+ 团队负责的工作内容是？+  今天的表现做怎么样的评价？
