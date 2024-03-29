---
title: "MySQL索引的工作原理"
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

]
---


索引是一种加快查询的数据结构，在 MySQL 中，索引的数据结构选择的是 B+Tree，至于 B+Tree 是什么以及为什么 MySQL 为什么选择 B+Tree 来作为索引，可以去查看公众号的前三篇文章。

- [索引数据结构之 B-Tree 与 B+Tree（上篇）](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2Fz_TNLqqJVwYKgb2kBadTwg)

- [索引数据结构之 B-Tree 与 B+Tree（下篇）](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FyLCqkrf1rvp6zJA6S-8sTQ)

- [MySQL 为什么不用数组、哈希表、二叉树等数据结构作为索引呢](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2F3zpqjT3cgYqYljgL-z0BKw)

今天主要来聊聊 MySQL 中索引的工作原理，这一部分的知识，在工作中经常被使用到，在面试中也几乎是必问的。所以，不管是面试造火箭，还是工作拧螺丝，掌握索引的工作原理，都是十分有必要的。

首先需要说明的是，本文的所有讨论均是基于 InnoDB 存储引擎为前提。

## 示例表

为了方便说明，我们先创建一个示例表。建表语句如下

```sql
CREATE TABLE user (
`id` BIGINT ( 11 ) NOT NULL AUTO_INCREMENT,
`name` VARCHAR ( 64 ) COMMENT '姓名',
`age` INT ( 4 ) COMMENT '年龄',
PRIMARY KEY ( `id` ),
INDEX ( NAME )
) ENGINE = INNODB COMMENT '用户表';

INSERT INTO `user` ( `name`, `age` ) VALUES ( 'AA', 30 ),( 'BB', 33 ),( 'CC', 31 ),( 'DD', 30 ),( 'EE', 29 )
```

在上面的 SQL 语句中，创建了一张 user 表，表中有三个字段，id 是主键，name 和 age 分别表示用户的姓名和年龄，同时还为字段 name 创建了一个普通索引。为了方便后面描述，因此还向表中插入了 5 条数据，由于主键 id 是自增的，所以这五行数据的 id 值分为是 1~5。

## 主键索引

主键索引又称之为聚簇索引（cluster index），它的特点是叶子结点中会存放当前主键所对应行的数据。什么意思呢？拿上面的例子来说明，在表 user 中，id 为主键索引，所以会有一棵 id 的索引树，在该索引树的叶子结点中，不仅存放了主键 id 的值，还存放了 name 和 value 的值。例如：在 id=1 这一行的数据中，name 和 age 的值为 AA 和 30，那么在索引树中，在 id=1 的结点处，存放的是(1,"AA",30)这三个值。id 索引树的示意图如下。

<img src="https://cdn.tkaid.com/img/1719c2d3191bcc77~tplv-t2oaga2asx-watermark.awebp" alt="图1" style="zoom: 33%;" />

下面看看这一条 SQL 语句的执行流程：

```sql
select * from user where id = 1;
```

该语句在 where 条件中加了 id=1 这个过滤条件，因此会使用到主键 id 的索引树。

1. 选择使用 id 主键索引树；
2. 找到 id 索引树的第一层结点（关键字 3、7 所在的结点），由于 where 条件中 id=1，1 小于 3，所以进入到关键字 3 的左子树中查找；
3. 进入到 id 索引树的第二层结点，第二层结点是叶子结点，叶子结点中存放了表的数据，并且存在 id=1 的关键字，所以将 R1 返回。（R1 表示的是 id=1 这一行的数据）。

## 回表

普通索引又称之为非聚簇索引，也叫做二级索引，它的特点是叶子结点中也会存放数据，与主键索引不同的是，普通索引中存放的数据只有主键的值，而非整行记录的数据。例如上面的示例表中，name 就是一个普通索引，它的索引树中，在叶子结点中存放的数据是主键 id 的值，示意图如下：

<img src="https://cdn.tkaid.com/img/1719c2da5a2c5d0a~tplv-t2oaga2asx-watermark.awebp" alt="图2" style="zoom: 33%;" />

下面看看这一条 SQL 语句的执行流程：

```sql
select * from user where name = 'BB';
```

该语句在 where 条件中加了 name='BB'这个过滤条件，由于我们在建表时为 name 字段创建了索引，因此会使用到 name 这棵索引树。另外，由于我们使用的是 select * ，也就是查询表中的所有字段的值，但是 name 索引树中只存有主键 id 的值，无法满足要查询所有字段的需求，而所有字段的数据都是存放在主键 id 索引树上的，因此在 name 索引树上查到主键 id 的值后，还需要根据查到的 id 值，再去主键索引树上查找这一行记录中其他字段的值，这个过程我们称之为回表。（从普通索引树回到主键索引树搜索的过程就叫做回表）。 所以上面的 SQL 语句的执行流程如下：

1. 选择使用 name 索引树；
2. 找到索引树的第一层结点，由于 where 条件中'BB'的值小于第一层结点中关键字'CC'的值，索引进入到关键字'CC'的左子树中查找；
3. 进入到第二层的叶子结点，找到关键字'BB'，由于叶子结点中存放了主键 id 的数据，所以返回'BB'中主键 id 的值 2；
4. 根据主键 id=2，再去主键 id 的索引树中查找，找到 id=2 所对应的数据 R2；
5. 在 name 索引树中继续向后查找，找到'BB'的下一个关键字'CC'，发现'CC'不等于 where 条件中的'BB'，所以结束查找。

## 覆盖索引

对于上面的第二个例子，由于 name='BB'的只有一条记录，因此只回了一次表，那如果有多条记录同时满足 name='BB'这个条件，那就得进行多次回表操作了。显然，回表次数越多，SQL 执行的越慢，那有什么办法能避免回表呢？答案就是覆盖索引。

覆盖索引究竟是个什么东西呢？在上面的第二个示例中，我们使用了 select * 来查询所有字段，那如果我们并不需要所有的字段呢，只需要 id 字段呢？例如 `select id from user where name = 'BB'`; 由于在 name 索引树的叶子结点中已经存有了主键 id 的值，所以 name 索引树能直接满足我们的查询要求，因此此时是不要回表操作，这种情况我们称之为覆盖索引。

覆盖索引能够显著的提升查询性能，因为它能明显减少大量的回表操作。覆盖索引是非常常用的一种 SQL 优化手段，使用起来也十分简单。我们在开发过程中，通常建议不要使用 select * 来查询数据，一方面是因为在数据量大时，select * 可能会返回好多无用字段，浪费网络资源；另一方面也是出于尽量使用覆盖索引的考虑。

## 联合索引与最左匹配原则

假如我们现在有个需求是要查询 user 表中，name='BB'的人的 name 和 age，我们的 SQL 需要这样写：

```sql
select name,age from user where name = 'BB';
```

显然，此时也会使用到 name 索引树，又因为 name 索引树中并没有存放 age 字段的信息，因此需要进行回表，回到主键 id 的索引树中取 age 字段的值。那么有什么方法能优化一下呢？让这次查询不需要进行回表。肯定有啊！使用覆盖索引啊。怎么用呢？

我们在创建 name 索引的时候，实际上创建的是单列索引（只选用了 name 这一列），而在 MySQL 中，我们是可以在创建索引时，选择多个列进行索引创建，这一类索引我们称之为联合索引。例如：我们现在为 name 字段和 age 字段创建一个联合索引，执行语句如下：

```sql
# 为了不影响测试，我们先将之前的name字段的索引删除
alter table user drop index `name`;
# 创建name、age的联合索引
alter table user add index(`name`,`age`);
```

这个时候，这个联合索引的索引树上，每个结点上存放的不仅仅只有 name 字段的值了，还有 age 字段的值，示意图如下：

<img src="https://cdn.tkaid.com/img/1719c2e161c18dac~tplv-t2oaga2asx-watermark.awebp" alt="图3" style="zoom: 33%;" />

那么这个时候，当我们 `select name,age from user where name = 'BB'` 时，由于需要的 name 字段和 age 字段在这棵联合索引树上已经存在了，所以这次查询不需要回表。

在使用联合索引时，索引的每一列只能做等值判断，因为 MySQL 会使用最左匹配原则进行匹配，也就是从索引最左边的列开始连续匹配，在碰到范围查找时会停止匹配，如遇到 like、>、<、between 等范围查找。可以结合下面三个示例来理解一下。

```sql
select name,age from user where name = 'BB' and age = 33; # 在使用联合索引时，会依次匹配name列和age列。
select name,age from user where name like 'B%' and age = 33; # 在使用联合索引时，当匹配到name这一列的时候，由于name使用了like范围查找，因此后面不会再匹配age这一列了。
select name,age from user where age = 33; # 在使用联合索引时，由于联合索引的最左列为name列，而我们在where条件中匹配的是age列，因此不满足最左匹配原则，所以该条SQL会进行该联合索引的全表扫描。
```

为什么 MySQL 要遵循最左匹配原则呢？这是因为 B+Tree 中，所有节点上的数据是有序的，当我们创建联合索引时，首先保证的是所有数据的第一列是有序的，然后再保证第二列、第三列以及后面的列有序。以上面的 user 表中的联合索引为例，在该索引树中，name 这一列在所有数据上是有序的，但是 age 这一列，却不是有序的，只有对于 name 相同的情况的下，age 才有序。当我们在查找数据时，如果碰到范围查找的时候，由于后面的列没法保证是有序的，所以不能再继续进行等值匹配，只能对后面的列进行全表扫描。

## 总结

本文主要讲了一条查询 SQL 语句是如何通过索引来查询数据的，以及什么是回表。在使用索引时，为了提升查询性能，可以通过创建合理的索引，使用覆盖索引来减少回表操作，从而达到提升查询性能的目的。最后，在联合索引的使用中，由于最左匹配原则，需要注意索引列的顺序，在创建联合索引时，需要考虑好如何安排索引内字段的顺序，以满足更多的查询场景，避免创建多个索引。

## 参考

[MySQL索引的工作原理](https://juejin.cn/post/6844904134433308685)

[索引数据结构之B-Tree与B+Tree（上篇）](https://mp.weixin.qq.com/s/z_TNLqqJVwYKgb2kBadTwg)

[索引数据结构之B-Tree与B+Tree（下篇）](https://mp.weixin.qq.com/s/yLCqkrf1rvp6zJA6S-8sTQ)

[MySQL为什么不用数组、哈希表、二叉树等数据结构作为索引呢](https://mp.weixin.qq.com/s/3zpqjT3cgYqYljgL-z0BKw)
