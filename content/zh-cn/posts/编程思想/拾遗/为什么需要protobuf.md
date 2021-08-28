---
title: "为什么需要protobuf"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"Other"
]
series : [
"Manual"
]
images : [
"images/center.png"
]
---

# 为什么需要protobuf

1. protobuf采用字节编码，而json, xml都是字符编码，字节编码更加节省空间
2. 采用了varint编码，进一步降低了编码后的空间大小

Varint就是一种对数字进行编码的方法，编码后二进制数据是不定长的，数值越小的数字使用的字节数越少。例如对于`int32_t`，采用Varint编码后需要1~5个bytes，小的数字使用1个byte，大的数字使用5个bytes。基于实际场景中小数字的使用远远多于大数字，因此通过Varint编码对于大部分场景都可以起到一个压缩的效果。
