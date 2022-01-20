---
title: "ZooKeeper的stat结构"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"ZK"
]
series : [
"Manual"
]
images : [

]
---



ZooKeeper命名空间中的每个znode都有一个与之关联的stat结构，类似于Unix/Linux文件系统中文件的stat结构。 znode的stat结构中的字段显示如下，各自的含义如下：

- `cZxid`：这是导致创建znode更改的事务ID。
- `mZxid`：这是最后修改znode更改的事务ID。
- `pZxid`：这是用于添加或删除子节点的znode更改的事务ID。
- `ctime`：表示从1970-01-01T00:00:00Z开始以毫秒为单位的znode创建时间。
- `mtime`：表示从1970-01-01T00:00:00Z开始以毫秒为单位的znode最近修改时间。
- `dataVersion`：表示对该znode的数据所做的更改次数。
- `cversion`：这表示对此znode的子节点进行的更改次数。
- `aclVersion`：表示对此znode的ACL进行更改的次数。
- `ephemeralOwner`：如果znode是ephemeral类型节点，则这是znode所有者的 session ID。 如果znode不是ephemeral节点，则该字段设置为零。
- `dataLength`：这是znode数据字段的长度。
- `numChildren`：这表示znode的子节点的数量。

在ZooKeeper Java shell中，可以使用`stat`或`ls`2命令查看znode的stat结构。 具体说明如下：

使用`stat`命令查看znode的`stat`结构：

```shell
[zk: localhost(CONNECTED) 0] stat /zookeeper
cZxid = 0x0
ctime = Thu Jan 01 05:30:00 IST 1970
mZxid = 0x0
mtime = Thu Jan 01 05:30:00 IST 1970
pZxid = 0x0
cversion = -1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 1
```

使用`ls2`命令查看znode的stat结构：

```shell
[zk: localhost(CONNECTED) 1] ls2 /zookeeper
[quota]
cZxid = 0x0
ctime = Thu Jan 01 05:30:00 IST 1970
mZxid = 0x0
mtime = Thu Jan 01 05:30:00 IST 1970
pZxid = 0x0
cversion = -1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 1
```

 

