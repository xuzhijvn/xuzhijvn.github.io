---
title: "k8s架构"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"云原生"
]
tags : [
"云原生",
"k8s"
]
series : [
"k8s"
]
images : [

]
---



<img src="http://cdn.tkaid.com/img/1583329564615.png" alt="img" style="zoom:50%;" />

Kubernetes 主要由以下几个核心组件组成:

- etcd 保存了整个集群的状态，就是一个数据库；
- apiserver 提供了资源操作的唯一入口，并提供认证、授权、访问控制、API 注册和发现等机制；
- controller manager 负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；
- scheduler 负责资源的调度，按照预定的调度策略将 Pod 调度到相应的机器上；
- kubelet 负责维护容器的生命周期，同时也负责 Volume（CSI）和网络（CNI）的管理；
- Container runtime 负责镜像管理以及 Pod 和容器的真正运行（CRI）；
- kube-proxy 负责为 Service 提供 cluster 内部的服务发现和负载均衡；

当然了除了上面的这些核心组件，还有一些推荐的插件：

- kube-dns 负责为整个集群提供 DNS 服务
- Ingress Controller 为服务提供外网入口
- Heapster 提供资源监控
- Dashboard 提供 GUI
