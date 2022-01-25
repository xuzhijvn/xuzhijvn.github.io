+++
title = "MQ实现延时消息"
description = ""
date = 2022-01-20T13:58:34+08:00
featured = false
comment = true
toc = true
reward = true
pinned = false
categories = [
"编程思想"
]
tags = [
"MQ"
]
series = [
"Manual"
]
images = [
]

+++

<!--more-->

本文分析一下目前比较主流的几款开源消息中间件对延时消息的支持情况以及实现方式。

### Kafka

原生Kafka默认是不支持延时消息的，需要开发者自己实现一层代理服务，比如发送端将消息发送到延时Topic，代理服务消费延时Topic的消息然后转存起来，代理服务通过一定的算法，计算延时消息所附带的延时时间是否到达，然后将延时消息取出来并发送到实际的Topic里面，消费端从实际的Topic里面进行消费。

[深入理解Kafka必知必会（3）](https://www.luozhiyun.com/archives/58)

[基于kafka实现延迟队列](https://zhuanlan.zhihu.com/p/365802989)

### RabbitMQ

RabbitMQ实现延时消息有两种方案，第一种是采用rabbitmq-delayed-message-exchange 插件实现，第二种则是利用DLX（Dead Letter Exchanges）+ TTL（消息存活时间）来间接实现。

第一种，在有许多延时消息量比较大时候会内存飙升。

第二种，大致的实现思路如下：

1. 创建一个普通队列delay_queue，为此队列设置死信交换机 (通过x-dead-letter-exchange参数) 和 RoutingKey (通过x-dead-letter-routing-key参数)，生产者将向delay_queue发送延时消息。
2. 创建步骤1中设置的死信交换机，同时创建一个目标队列 target_queue，并使用步骤1中设置的RoutingKey将两者绑定起来。消费者将从target_queue里面消费延时消息。
3. 设置消息的存活时间TTL，可以在步骤1中设置到队列级别delay_queue的消息存活时间（不灵活），或者在发送消息时动态设置消息级别的存活时间（后面的消息要等前面的消息成为死信后才会“死亡”）。

[一文带你搞定RabbitMQ死信队列](https://www.cnblogs.com/mfrank/p/11184929.html)

[一文带你搞定RabbitMQ延迟队列](https://www.cnblogs.com/mfrank/p/11260355.html)

### RocketMQ

开源RocketMQ支持延迟消息，但是不支持秒级精度。默认支持18个level的延迟消息，这是通过broker端的messageDelayLevel配置项确定的
`messageDelayLevel=1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h`

消息队列服务在启动时，会创建一个内部topic：SCHEDULE_TOPIC_XXXX，根据延迟level的个数，创建对应数量的队列。生产者发送消息时可以设置延时等级，示例代码:

```java
Message msg=new Message();
msg.setTopic("TopicA");
msg.setBody("this is a delay message".getBytes());
//设置延迟level为5，对应延迟1分钟
msg.setDelayTimeLevel(5);
producer.send(msg);
```

发送的消息会暂存在Broker对应的内部topic中，再通过定时任务从内部topic中拉取数据，如果延迟时间到了，就会把消息转发到目标topic下，消费者从目标topic消费消息。

[消息队列——延时消息应用解析及实践](https://developer.aliyun.com/article/780050)
