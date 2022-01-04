+++
title = "RabbitMQ清空队列的几种方式"
description = ""
date = 2022-01-04T10:04:18+08:00
featured = false
comment = true
toc = true
reward = true
pinned = false
categories = [
  "编程思想"
]
tags = [
  "RabbitMQ, MQ"
]
series = [
  "Manual"
]
images = []

+++

🔢

这里列举几种清空RabbitMQ队列的方案

<!--more-->

- purge

```shell
rabbitmqctl purge_queue queue001
```

> 该方式能删除所有ready的消息，对于unacked消息无法删除。如果需要删除unacked消息，需要将该队列上的所有消费者停止，unacked消息会自动变为ready消息，此时通过purge_queue命令可以删除。

- delete删除队列，然后重建

删除：

```shell
rabbitmqctl delete_queue queue001
```

- reset

```shell
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl start_app
```

> 注意此方式，会同时清除一些配置信息，需要慎用。

- 消费到消息，自动或手动ack



## 参考

[rabbitmq 如何删除队列中的消息](https://blog.csdn.net/fly_leopard/article/details/102599532)

[RabbitMQ手册之rabbitmqctl](https://www.jianshu.com/p/61a90fba1d2a)

[RabbitMQ - Delete Queue using the rabbitmqctl command](http://www.freekb.net/Article?id=2798)
