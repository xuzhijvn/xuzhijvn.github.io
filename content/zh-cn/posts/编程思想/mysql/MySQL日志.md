---
title: "MySQL日志"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"MySQL"
]
series : [
"Manual"
]
images : [
"images/center.png"
]
---

[comment]: <> (# MySQL日志)



日志是`mysql`数据库的重要组成部分，记录着数据库运行期间各种状态信息。`mysql`日志主要包括`错误日志`、`查询日志`、`慢查询日志`、`事务日志`、`二进制日志`几大类。作为开发，我们重点需要关注的是二进制日志(`binlog`)和事务日志(包括`redo log`和`undo log`)，本文接下来会详细介绍这三种日志。其他几种日志见 👉 [[玩转MySQL之八]MySQL日志分类及简介](https://zhuanlan.zhihu.com/p/58011817)

## 1. binlog

`binlog`用于记录数据库执行的写入性操作(不包括查询)信息，以二进制的形式保存在磁盘中。`binlog`是`mysql`的逻辑日志，并且由`Server`层进行记录，使用任何存储引擎的`mysql`数据库都会记录`binlog`日志。

> 逻辑日志：**可以简单理解为记录的就是sql语句**。

> 物理日志：**因为`mysql`数据最终是保存在数据页中的，物理日志记录的就是数据页变更**。

`binlog`是通过追加的方式进行写入的，可以通过`max_binlog_size`参数设置每个`binlog`文件的大小，当文件大小达到给定值之后，会生成新的文件来保存日志。

### 1.1 binlog使用场景

在实际应用中，`binlog`的主要使用场景有两个，分别是**主从复制**和**数据恢复**。

1. **主从复制**：在`Master`端开启`binlog`，然后将`binlog`发送到各个`Slave`端，`Slave`端重放`binlog`从而达到主从数据一致。
2. **数据恢复**：通过使用`mysqlbinlog`工具来恢复数据。

### 1.2 binlog刷盘时机

对于`InnoDB`存储引擎而言，只有在事务提交时才会记录`biglog`，此时记录还在内存中，那么`biglog`是什么时候刷到磁盘中的呢？`mysql`通过`sync_binlog`参数控制`biglog`的刷盘时机，取值范围是`0-N`：

- 0：不去强制要求，由系统自行判断何时写入磁盘；
- 1：每次`commit`的时候都要将`binlog`写入磁盘；
- N：每N个事务，才会将`binlog`写入磁盘。

从上面可以看出，`sync_binlog`最安全的是设置是`1`，这也是`MySQL 5.7.7`之后版本的默认值。但是设置一个大一些的值可以提升数据库性能，因此实际情况下也可以将值适当调大，牺牲一定的一致性来获取更好的性能。

### 1.3 binlog日志格式

`binlog`日志有三种格式，分别为`STATMENT`、`ROW`和`MIXED`。

> 在 `MySQL 5.7.7`之前，默认的格式是`STATEMENT`，`MySQL 5.7.7`之后，默认值是`ROW`。日志格式通过`binlog-format`指定。

- `STATMENT` **基于`SQL`语句的复制(`statement-based replication, SBR`)，每一条会修改数据的sql语句会记录到`binlog`中**。 优点：**不需要记录每一行的变化，减少了`binlog`日志量，节约了`IO`, 从而提高了性能**； 缺点：**在某些情况下会导致主从数据不一致，比如执行`sysdate()`、`slepp()`等**。
- `ROW` **基于行的复制(`row-based replication, RBR`)，不记录每条sql语句的上下文信息，仅需记录哪条数据被修改了**。 优点：**不会出现某些特定情况下的存储过程、或function、或trigger的调用和触发无法被正确复制的问题**； 缺点：**会产生大量的日志，尤其是`alter table`的时候会让日志暴涨**
- `MIXED` **基于`STATMENT`和`ROW`两种模式的混合复制(`mixed-based replication, MBR`)，一般的复制使用`STATEMENT`模式保存`binlog`，对于`STATEMENT`模式无法复制的操作使用`ROW`模式保存`binlog`**

## 2. redo log

### 2.1 为什么需要redo log

我们都知道，事务的四大特性里面有一个是**持久性**，具体来说就是**只要事务提交成功，那么对数据库做的修改就被永久保存下来了，不可能因为任何原因再回到原来的状态**。那么`mysql`是如何保证持久性的呢？最简单的做法是在每次事务提交的时候，将该事务涉及修改的数据页全部刷新到磁盘中。但是这么做会有严重的性能问题，主要体现在两个方面：

1. 因为`Innodb`是以`页`为单位进行磁盘交互的，而一个事务很可能只修改一个数据页里面的几个字节，这个时候将完整的数据页刷到磁盘的话，太浪费资源了！
2. 一个事务可能涉及修改多个数据页，并且这些数据页在物理上并不连续，使用随机IO写入性能太差！

因此`mysql`设计了`redo log`，**具体来说就是只记录事务对数据页做了哪些修改**，这样就能完美地解决性能问题了(相对而言文件更小并且是顺序IO)。[MySQL实战45讲](https://time.geekbang.org/column/article/68633)中将`redo log`比作临时记账的`粉板`，`实际数据页`比作`账本`。

### 2.2 redo log基本概念

`redo log`包括两部分：一个是内存中的日志缓冲(`redo log buffer`)，另一个是磁盘上的日志文件(`redo log file`)。`mysql`每执行一条`DML`语句，先将记录写入`redo log buffer`，后续某个时间点再一次性将多个操作记录写到`redo log file`。这种**先写日志，再写磁盘**的技术就是`MySQL`里经常说到的`WAL(Write-Ahead Logging)` 技术。

在计算机操作系统中，用户空间(`user space`)下的缓冲区数据一般情况下是无法直接写入磁盘的，中间必须经过操作系统内核空间(`kernel space`)缓冲区(`OS Buffer`)。因此，`redo log buffer`写入`redo log file`实际上是先写入`OS Buffer`，然后再通过系统调用`fsync()`将其刷到`redo log file`中，过程如下：

![从redo log buffer写日志到磁盘的redo log file中](https://picgo.6and.ltd/img/v2-44c0e78a1222853dc2a99e596cf3fa13_r.jpg)

`mysql`支持三种将`redo log buffer`写入`redo log file`的时机，可以通过`innodb_flush_log_at_trx_commit`参数配置，各参数值含义如下：

| 参数值              | 含义                                                         |
| ------------------- | ------------------------------------------------------------ |
| 0（延迟写）         | 事务提交时不会将`redo log buffer`中日志写入到`os buffer`，而是每秒写入`os buffer`并调用`fsync()`写入到`redo log file`中。也就是说设置为0时是(大约)每秒刷新写入到磁盘中的，当系统崩溃，会丢失1秒钟的数据。 |
| 1（实时写，实时刷） | 事务每次提交都会将`redo log buffer`中的日志写入`os buffer`并调用`fsync()`刷到`redo log file`中。这种方式即使系统崩溃也不会丢失任何数据，但是因为每次提交都写入磁盘，IO的性能较差。 |
| 2（实时写，延迟刷） | 每次提交都仅写入到`os buffer`，然后是每秒调用`fsync()`将`os buffer`中的日志写入到`redo log file`。 |

![自定义在commit时如何将log buffer中的日志刷log file中](https://picgo.6and.ltd/img/v2-6322074d51c4d1507be3c3159ab583bc_r-20210819145000945-20210819145044199.jpg)

### 2.3 redo log记录形式

前面说过，`redo log`实际上记录数据页的变更，而这种变更记录是没必要全部保存，因此`redo log`实现上采用了大小固定，循环写入的方式，当写到结尾时，会回到开头循环写日志。如下图： 

<img src="https://picgo.6and.ltd/img/image-20210819145133194.png" alt="redo log记录形式" style="zoom: 33%;" />

同时我们很容易得知，**在innodb中，既有`redo log`需要刷盘，还有`数据页`也需要刷盘，`redo log`存在的意义主要就是降低对`数据页`刷盘的要求**。在上图中，`write pos`表示`redo log`当前记录的`LSN`(逻辑序列号)位置，`check point`表示**数据页更改记录**刷盘后对应`redo log`所处的`LSN`(逻辑序列号)位置。`write pos`到`check point`之间的部分是`redo log`空着的部分，用于记录新的记录；`check point`到`write pos`之间是`redo log`待落盘的数据页更改记录。当`write pos`追上`check point`时，会先推动`check point`向前移动，空出位置再记录新的日志。

启动`innodb`的时候，不管上次是正常关闭还是异常关闭，总是会进行恢复操作。因为`redo log`记录的是数据页的物理变化，因此恢复的时候速度比逻辑日志(如`binlog`)要快很多。 重启`innodb`时，首先会检查磁盘中数据页的`LSN`，如果数据页的`LSN`小于日志中的`LSN`，则会从`checkpoint`开始恢复。 还有一种情况，在宕机前正处于`checkpoint`的刷盘过程，且数据页的刷盘进度超过了日志页的刷盘进度，此时会出现数据页中记录的`LSN`大于日志中的`LSN`，这时超出日志进度的部分将不会重做，因为这本身就表示已经做过的事情，无需再重做。

### 2.4 redo log与binlog区别

|          | redo log                                                     | binlog                                                       |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 文件大小 | `redo log`的大小是固定的。                                   | `binlog`可通过配置参数`max_binlog_size`设置每个`binlog`文件的大小。 |
| 实现方式 | `redo log`是`InnoDB`引擎层实现的，并不是所有引擎都有。       | `binlog`是`Server`层实现的，所有引擎都可以使用 `binlog`日志  |
| 记录方式 | redo log 采用循环写的方式记录，当写到结尾时，会回到开头循环写日志。 | binlog 通过追加的方式记录，当文件大小大于给定值后，后续的日志会记录到新的文件上 |
| 适用场景 | `redo log`适用于崩溃恢复(crash-safe)                         | `binlog`适用于主从复制和数据恢复                             |

由`binlog`和`redo log`的区别可知：`binlog`日志只用于归档，只依靠`binlog`是没有`crash-safe`能力的。但只有`redo log`也不行，因为`redo log`是`InnoDB`特有的，且日志上的记录落盘后会被覆盖掉。因此需要`binlog`和`redo log`二者同时记录，才能保证当数据库发生宕机重启时，数据不会丢失。

> 问：binlog为什么没有crash_safe的能力呢？binlog日志也记录了所有的操作。
>
> 答：不考虑mysql现有的实现，假如现在重新设计mysql，只用一个binlog是否可以实现cash_safe能力呢？答案是可以的，只不过binlog中也要加入checkpoint，数据库故障重启后，binlog checkpoint之后的sql都重放一遍。但是这样做让binlog耦合的功能太多。

## 3. undo log

数据库事务四大特性中有一个是**原子性**，具体来说就是 **原子性是指对数据库的一系列操作，要么全部成功，要么全部失败，不可能出现部分成功的情况**。实际上，**原子性**底层就是通过`undo log`实现的。`undo log`主要记录了数据的逻辑变化，比如一条`INSERT`语句，对应一条`DELETE`的`undo log`，对于每个`UPDATE`语句，对应一条相反的`UPDATE`的`undo log`，这样在发生错误时，就能回滚到事务之前的数据状态。同时，`undo log`也是`MVCC`(多版本并发控制)实现的关键，这部分内容在[面试中的老大难-mysql事务和锁，一次性讲清楚！](https://juejin.cn/post/6855129007336521741)中有介绍，不再赘述。



## 附件

### MySQL 的逻辑架构图：

<img src="https://picgo.6and.ltd/img/0d2070e8f84c4801adbfa03bda1f98d9-20210819150540455.png" alt="MySQL 的逻辑架构图" style="zoom: 50%;" />

mysql8.0之后删除了查询缓存模块

### update 语句执行流程：

<img src="https://picgo.6and.ltd/img/20201124231528159.png" alt="update 语句执行流程" style="zoom:50%;" />

redo log的写入分成两个步骤prepare和commit，这就是**两阶段提交**

> 问：两阶段提交中，MySQL异常重启（crash），是如何保证数据完整性的？
>
> 答：
>
> 1. 在上图时刻A中，也就是写入redo log处于prepare阶段以后、写binlog之前，发生了崩溃（crash）：由于此时binlog还没写，redo log也还没提交（commit），所以崩溃恢复的时候，这个事务会回滚。这时候，binlog还没写，所以也不会传到备库，数据一致；
> 2. 在上图时刻B中，也就是写完binlog之后，发生crash：如果redo log里面的事务有commit标识（事务是完整的）则直接提交；如果redo log里面的事务只有prepare没有commit，则判断对应的事务在binlog是否存在并完整，完整则提交事务，否则回滚事务；
> 3. MySQL怎么知道binlog是完整的：一个事务的binlog是有完整格式的，statement格式（记录sql语句），最后会有个COMMIT; row格式（记录行的内容，记两条，更新前和更新后都有），最后会有个XID event
> 4. redo log和binlog是怎么关联起来的：他们有一个共同的数据字段XID。崩溃恢复的时候，会按顺序扫描redo log；上述第二点中的崩溃恢复场景（redo log里面的事务只有完整的prepare而没有commit），那么mysql就会拿着XID去binlog找是否存在对应的完整事务；
>    



## 参考

[必须了解的mysql三大日志-binlog、redo log和undo log](https://juejin.cn/post/6860252224930070536)

[[玩转MySQL之八]MySQL日志分类及简介](https://zhuanlan.zhihu.com/p/58011817)

[01 | 基础架构：一条SQL查询语句是如何执行的？](https://time.geekbang.org/column/article/68319)

[02 | 日志系统：一条SQL更新语句是如何执行的？](https://time.geekbang.org/column/article/68633)

[为什么有binlog还要redo log](https://blog.csdn.net/ab1024249403/article/details/110099571)

