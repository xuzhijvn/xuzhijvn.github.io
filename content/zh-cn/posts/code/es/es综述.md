---
title: "ES综述"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"ES"
]
series : [
"Manual"
]
images : [

]
---

[comment]: <> (# ES综述)

## 1. ES 的分布式架构原理

ElasticSearch 设计的理念就是分布式搜索引擎，底层其实还是基于 lucene 的。核心思想就是在多台机器上启动多个 ES 进程实例，组成了一个 ES 集群。

默认节点会去加入一个名称为 `elasticsearch` 的集群。如果直接启动一堆节点，那么它们会自动组成一个 elasticsearch 集群，当然一个节点也可以组成 elasticsearch 集群。

### 1.1 ES数据结构

ES 中存储数据的**基本单位是索引**，比如说你现在要在 ES 中存储一些订单数据，你就应该在 ES 中创建一个索引 `order_idx` ，所有的订单数据就都写到这个索引里面去，一个索引差不多就是相当于是 mysql 里的一张表。

```
index -> type -> mapping -> document -> field。
```

### 1.2 ES高可用

类似kafka将一个topic分成多个partition，es将一个index分成多个shard。

> 每个 shard 存储部分数据。拆分多个 shard 是有好处的，一是**支持横向扩展**，比如你数据量是 3T，3 个 shard，每个 shard 就 1T 的数据，若现在数据量增加到 4T，怎么扩展，很简单，重新建一个有 4 个 shard 的索引，将数据导进去；二是**提高性能**，数据分布在多个 shard，即多台服务器上，所有的操作，都会在多台机器上并行分布式执行，提高了吞吐量和性能。

kafka的每个partition有多个replica副本，所有副本选出一个leader（依赖zookeeper完成选举）负责所有的读写请求。

ES 的每个shard也可以配置多个replica副本，每个 shard 都有一个 `primary shard` ，负责写入数据，但是还有几个 `replica shard` 。 `primary shard` 写入数据之后，会将数据同步到其他几个 `replica shard` 上去。

> primary shard（建立索引时一次设置，不能修改，默认 5 个），replica shard（随时修改数量，默认 1 个），默认每个索引 10 个 shard，5 个 primary shard，5 个 replica shard，最小的高可用配置，是 2 台服务器。

与kafka partition副本不同的是，es不仅 `primary shard` 能读写，`replica shard` 也能读。

![es-cluster](https://cdn.tkaid.com/img/es-cluster.png)

ES 集群多个节点，会自动选举一个节点为 master 节点，这个 master 节点其实就是干一些管理的工作的，比如维护索引元数据、负责切换 primary shard 和 replica shard 身份等。要是 master 节点宕机了，那么会重新选举一个节点为 master 节点。

与kafka依赖zookeeper完成 `选举主节点服务器(controller)`、  `选举partition leader`方式不同的是，es依赖自己的实现（ZenDiscovery）完成集群`master 节点选举`、`primary shard切换` 不依赖第三方组件。

> [一致性算法研究(三)Kafka](https://www.wangjunfei.com/2020/02/26/%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%E7%A0%94%E7%A9%B6-%E4%B8%89-Kafka/)
>
> 但是，中间过程中的机器宕机等情况时，怎么来保证一致性呢？ 这个问题，kafka交给了zookeeper来解决。kafka集群启动时，会依靠zookeeper来选举一个集群的主节点服务器(controller)，controller来管理整个集群，比如队列的创建、删除，队列分区的选主，如前边所说的，topicA有3个分区，每个分区有3个副本(replica)，在创建每个分区时，kafka会使用zookeeper来选举一个分区副本的主(leader)。
>
> 一个集群内有上千个队列，每个队列都有数十个甚至数百个分区，每个分区又会有若干个副本，多个副本之间谁是leader副本？这些信息称之为metadata，kafka全部储存在了zookeeper上。
>
> 所以机器宕机时，controller和leader replica的选举任务都交给了zookeeper来完成，那么这时集群内机器之间的一致性，分区副本之间的一致性，这些共识问题都交给了zookeeper来解决。有了zookeeper，集群内每个机器不会对谁是controller再产生分歧，有了controller，一个集群就一个统一的“负责人”。同时，每个分区的副本之间使用zookeeper来选主，也会产生一个“负责人”，所有机器，多个副本之间都交由这些负责人来协调，不会再产生分歧，而且不会脑裂。



> [Elasticsearch分布式一致性原理剖析(一)-节点篇](https://zhuanlan.zhihu.com/p/34830403)
>
> 那为什么ES不使用Zookeeper呢，大概是官方开发觉得增加Zookeeper依赖后会多依赖一个组件，使集群部署变得更复杂，用户在运维时需要多运维一个Zookeeper。
>
> 那么在自主实现这条路上，还有什么别的算法选择吗？当然有的，比如raft
>
> raft从正确性上看肯定是更好的选择，而ES的选举算法经过几次bug fix也越来越像raft。当然，在ES最早开发时还没有raft（es的前身compass于2004开源，2010进行重构并改名为es，raft算法2012才发表😔），而未来ES如果继续沿着这个方向走很可能最终就变成一个raft实现。



## 2. ES 的工作原理

### 2.1 底层 lucene

简单来说，lucene 就是一个 jar 包，里面包含了封装好的各种建立倒排索引的算法代码。我们用 Java 开发的时候，引入 lucene jar，然后基于 lucene 的 api 去开发就可以了。

通过 lucene，我们可以将已有的数据建立索引，lucene 会在本地磁盘上面，给我们组织索引的数据结构。

### 2.2 倒排索引

在搜索引擎中，每个文档都有一个对应的文档 ID，文档内容被表示为一系列关键词的集合。例如，文档 1 经过分词，提取了 20 个关键词，每个关键词都会记录它在文档中出现的次数和出现位置。

那么，倒排索引就是**关键词到文档** ID 的映射，每个关键词都对应着一系列的文件，这些文件中都出现了关键词。

举个栗子。

有以下文档：

| DocId | Doc                                            |
| ----- | ---------------------------------------------- |
| 1     | 谷歌地图之父跳槽 Facebook                      |
| 2     | 谷歌地图之父加盟 Facebook                      |
| 3     | 谷歌地图创始人拉斯离开谷歌加盟 Facebook        |
| 4     | 谷歌地图之父跳槽 Facebook 与 Wave 项目取消有关 |
| 5     | 谷歌地图之父拉斯加盟社交网站 Facebook          |

对文档进行分词之后，得到以下**倒排索引**。

| WordId | Word     | DocIds        |
| ------ | -------- | ------------- |
| 1      | 谷歌     | 1, 2, 3, 4, 5 |
| 2      | 地图     | 1, 2, 3, 4, 5 |
| 3      | 之父     | 1, 2, 4, 5    |
| 4      | 跳槽     | 1, 4          |
| 5      | Facebook | 1, 2, 3, 4, 5 |
| 6      | 加盟     | 2, 3, 5       |
| 7      | 创始人   | 3             |
| 8      | 拉斯     | 3, 5          |
| 9      | 离开     | 3             |
| 10     | 与       | 4             |
| ..     | ..       | ..            |

另外，实用的倒排索引还可以记录更多的信息，比如文档频率信息，表示在文档集合中有多少个文档包含某个单词。

那么，有了倒排索引，搜索引擎可以很方便地响应用户的查询。比如用户输入查询 `Facebook` ，搜索系统查找倒排索引，从中读出包含这个单词的文档，这些文档就是提供给用户的搜索结果。

要注意倒排索引的两个重要细节：

- 倒排索引中的所有词项对应一个或多个文档；
- 倒排索引中的词项**根据字典顺序升序排列**

> 上面只是一个简单的栗子，并没有严格按照字典顺序升序排列。



## 3.ES 读写检索数据的过程

### 3.1 ES 写数据过程

- 客户端选择一个 node 发送请求过去，这个 node 就是 `coordinating node` （协调节点）。
- `coordinating node` 对 document 进行**路由**，将请求转发给对应的 node（有 primary shard）。
- 实际的 node 上的 `primary shard` 处理请求，然后将数据同步到 `replica node` 。
- `coordinating node` 如果发现 `primary node` 和所有 `replica node` 都搞定之后，就返回响应结果给客户端。

![es-write](https://cdn.tkaid.com/img/es-write.png)

### 3.2 ES 读数据过程

```http
GET my-index/_doc/0
```



![查询流程](https://cdn.tkaid.com/img/209c63fb760b19cb4e9c745c1e795045.png)



1. Client 将请求发送到任意节点 node，此时 node 节点就是**协调节点**（coordinating node）。
2. 协调节点对 id 进行路由，从而判断该数据在哪个 shard。
3. 在 primary shard 和 replica shard 之间 随机选择一个，请求获取 doc。
4. 接收请求的节点会将数据返回给**协调节点**，协调节点会将数据返回给 Client。

> 可以通过 preference 参数指定执行操作的节点或分片。默认为随机。

### 3.3 ES 检索数据过程

```http
GET /my-index/_search 
```

es 最强大的是做全文检索，就是比如你有三条数据：

```
java真好玩儿啊
java好难学啊
j2ee特别牛
```

你根据 `java` 关键词来搜索，将包含 `java` 的 `document` 给搜索出来。es 就会给你返回：java 真好玩儿啊，java 好难学啊。

- 客户端发送请求到一个 `coordinate node` 。
- 协调节点将搜索请求转发到**所有**的 shard 对应的 `primary shard` 或 `replica shard` ，都可以。
- query phase：每个 shard 将自己的搜索结果（其实就是一些 `doc id` ）返回给协调节点，由协调节点进行数据的合并、排序、分页等操作，产出最终结果。
- fetch phase：接着由协调节点根据 `doc id` 去各个节点上**拉取实际**的 `document` 数据，最终返回给客户端。

> 写请求是写入 primary shard，然后同步给所有的 replica shard；读请求可以从 primary shard 或 replica shard 读取，采用的是随机轮询算法。



## 4. 优化ES提升检索效率



### 4.1 filesystem cache

`filesystem cache` 即`文件系统cache`，由文件系统进行热点数据缓存，不受进程自主控制。关于cache的详细介绍请 👉 [cache和buffer](https://huolala.gitbook.io/code/ji-suan-ji-ji-chu/cao-zuo-xi-tong/cache-he-buffer)

你往 es 里写的数据，实际上都写到磁盘文件里去了，**查询的时候**，操作系统会将磁盘文件里的数据自动缓存到 `filesystem cache` 里面去。

![es-search-process](https://cdn.tkaid.com/img/es-search-process.png)

归根结底，你要让 es 性能要好，最佳的情况下，就是你的机器的内存，至少可以容纳你的总数据量的一半。

根据我们自己的生产环境实践经验，最佳的情况下，是仅仅在 es 中就存少量的数据，就是你要**用来搜索的那些索引**，如果内存留给 `filesystem cache` 的是 100G，那么你就将索引数据控制在 `100G` 以内，这样的话，你的数据几乎全部走内存来搜索，性能非常之高，一般可以在 1 秒以内。

比如说你现在有一行数据。 `id,name,age ....` 30 个字段。但是你现在搜索，只需要根据 `id,name,age` 三个字段来搜索。如果你傻乎乎往 es 里写入一行数据所有的字段，就会导致说 `90%` 的数据是不用来搜索的，结果硬是占据了 es 机器上的 `filesystem cache` 的空间，单条数据的数据量越大，就会导致 `filesystem cahce` 能缓存的数据就越少。其实，仅仅写入 es 中要用来检索的**少数几个字段**就可以了，比如说就写入 es `id,name,age` 三个字段，然后你可以把其他的字段数据存在 mysql/hbase 里，我们一般是建议用 `es + hbase` 这么一个架构。

hbase 的特点是**适用于海量数据的在线存储**，就是对 hbase 可以写入海量数据，但是不要做复杂的搜索，做很简单的一些根据 id 或者范围进行查询的这么一个操作就可以了。从 es 中根据 name 和 age 去搜索，拿到的结果可能就 20 个 `doc id` ，然后根据 `doc id` 到 hbase 里去查询每个 `doc id` 对应的**完整的数据**，给查出来，再返回给前端。

写入 es 的数据最好小于等于，或者是略微大于 es 的 filesystem cache 的内存容量。然后你从 es 检索可能就花费 20ms，然后再根据 es 返回的 id 去 hbase 里查询，查 20 条数据，可能也就耗费个 30ms，可能你原来那么玩儿，1T 数据都放 es，会每次查询都是 5~10s，现在可能性能就会很高，每次查询就是 50ms。

### 4.2 数据预热

对于那些你觉得比较热的、经常会有人访问的数据，最好**做一个专门的缓存预热子系统**，就是对热数据每隔一段时间，就提前访问一下，让数据进入 `filesystem cache` 里面去。这样下次别人访问的时候，性能一定会好很多。

### 4.3 冷热分离

假设你有 6 台机器，2 个索引，一个放冷数据，一个放热数据，每个索引 3 个 shard。3 台机器放热数据 index，另外 3 台机器放冷数据 index。然后这样的话，你大量的时间是在访问热数据 index，热数据可能就占总数据量的 10%，此时数据量很少，几乎全都保留在 `filesystem cache` 里面了，就可以确保热数据的访问性能是很高的。但是对于冷数据而言，是在别的 index 里的，跟热数据 index 不在相同的机器上，大家互相之间都没什么联系了。如果有人访问冷数据，可能大量数据是在磁盘上的，此时性能差点，就 10% 的人去访问冷数据，90% 的人在访问热数据，也无所谓了。

### 4.4 document 模型设计

最好是先在 Java 系统里就完成关联，将关联好的数据直接写入 es 中。搜索的时候，就不需要利用 es 的搜索语法来完成 join 之类的关联搜索了。

document 模型设计是非常重要的，很多操作，不要在搜索的时候才想去执行各种复杂的乱七八糟的操作。es 能支持的操作就那么多，不要考虑用 es 做一些它不好操作的事情。如果真的有那种操作，尽量在 document 模型设计的时候，写入的时候就完成。另外对于一些太复杂的操作，比如 join/nested/parent-child 搜索都要尽量避免，性能都很差的。

### 4.5 分页性能优化

es 的分页是较坑的，为啥呢？举个例子吧，假如你每页是 10 条数据，你现在要查询第 100 页，实际上是会把每个 shard 上存储的前 1000 条数据都查到一个协调节点上，如果你有个 5 个 shard，那么就有 5000 条数据，接着协调节点对这 5000 条数据进行一些合并、处理，再获取到最终第 100 页的 10 条数据。

分布式的，你要查第 100 页的 10 条数据，不可能说从 5 个 shard，每个 shard 就查 2 条数据，最后到协调节点合并成 10 条数据吧？你**必须**得从每个 shard 都查 1000 条数据过来，然后根据你的需求进行排序、筛选等等操作，最后再次分页，拿到里面第 100 页的数据。你翻页的时候，翻的越深，每个 shard 返回的数据就越多，而且协调节点处理的时间越长，非常坑爹。所以用 es 做分页的时候，你会发现越翻到后面，就越是慢。

我们之前也是遇到过这个问题，用 es 作分页，前几页就几十毫秒，翻到 10 页或者几十页的时候，基本上就要 5~10 秒才能查出来一页数据了。

有什么解决方案吗？

**不允许深度分页（默认深度分页性能很差）**

跟产品经理说，你系统不允许翻那么深的页，默认翻的越深，性能就越差。

**类似于 app 里的推荐商品不断下拉出来一页一页的**

`scroll api` 或者 `search_after`

## 参考

**基础：**

[doocs/advanced-java](doocs/advanced-java)

[【Elasticsearch 技术分享】—— ES 查询检索数据的过程，是什么样子的？](https://xie.infoq.cn/article/7f84ceaca3e1537eab40b853d)

[全文搜索引擎 Elasticsearch 入门教程](https://www.ruanyifeng.com/blog/2017/08/elasticsearch.html)

**应用场景：**

[七个生产案例告诉你BATJ为何选择ElasticSearch！应用场景和优势！](https://www.cnblogs.com/liuyanling/p/13023251.html)

**项目实战：**

[Elasticsearch快速入门，掌握这些刚刚好！](https://mp.weixin.qq.com/s/cohWZy_eUOUqbmUxhXzzNA)

[mall整合Elasticsearch实现商品搜索](https://mp.weixin.qq.com/s/op7CTQS5dGOKv5BvRbeFow)

[Elasticsearch项目实战，商品搜索功能设计与实现！](https://juejin.cn/post/6844904126321524744)

