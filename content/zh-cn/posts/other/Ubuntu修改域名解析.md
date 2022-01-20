---
title: "Ubuntu修改域名解析"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"其他"
]
tags : [
"Other"
]
series : [
"Manual"
]
images : [

]
---

[comment]: <> (# Ubuntu修改域名解析)

1、vim /etc/resolv.conf

```shell
nameserver 8.8.8.8 
nameserver 114.114.114.114 
nameserver 1.2.4.8
```





2、vim /etc/network/interfaces

```shell
dns-nameserver 8.8.8.8 
dns-nameserver 114.114.114.114
dns-nameserver 1.2.4.8
```



3、/etc/init.d/networking restart
