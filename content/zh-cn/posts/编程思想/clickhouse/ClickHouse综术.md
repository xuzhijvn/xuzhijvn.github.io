---
title: "ClickHouse综述"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"ClickHouse"
]
series : [
"Manual"
]
images : [
"images/center.png"
]
---

[comment]: <> (# ClickHouse综述)

## 列式存储优势

- 当查询语句只涉及部分列时，只需要扫描相关的列

  ![大幅降低磁盘IO](https://picgo.6and.ltd/img/3412665-fb1b4b09b70815f2.jpg)

- 每一列的数据都是相同类型的，彼此间相关性更大，对列数据压缩的效率较高



## 参考

[“行式存储”和“列式存储”的区别](https://www.jianshu.com/p/3d3950c9fb06)

[什么是ClickHouse？](https://clickhouse.tech/docs/zh/)
