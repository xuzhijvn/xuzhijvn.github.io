---
title: "iptables: No chain/target/match by that name."
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

iptables: No chain/target/match by that name.

重启redis镜像的时候报错如下：

```shell
ERROR: for redis Cannot start service redis: driver failed programming external connectivity on endpoint redis (f5211771e5a0ee705edb72f8a8dfbca2503456ab0e8330a32932b029a7c0568d): (iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 6379 -j DNAT --to-destination 192.168.80.2:6379 ! -i br-fbefac0273aa: iptables: No chain/target/match by that name.
```

原因：

docker 服务启动的时候，docker服务会向iptables注册一个链，以便让docker服务管理的containner所暴露的端口之间进行通信。通过命令`iptables -L`可以查看iptables 链。如果你删除了iptables中的docker链，或者iptables的规则被丢失了（例如重启firewalld，我就是使用了`systemctl stop iptables`导致链丢失），docker就会报这个错误。

解决办法：

`systemctl restart docker `重启docker服务，之后，正确的iptables规则就会被创建出来。

参考：

[Docker 启动时报错：iptables:No chain/target/match by the name](https://blog.csdn.net/a1010256340/article/details/79986959)
