---
title: "k8s接口"
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
"images/center.png"
]
---



<img src="https://picgo.6and.ltd/img/8ee9f2fa987eccb490cfaa91c6484f67.png" alt="img" style="zoom: 43%;" />

几种接口解释：

- `CRI（Container Runtime Interface）`。容器运行时接口。Kubernetes项目并不关心你部署的是什么容器运行时、使用的什么技术实现，只要你的这个容器运行时能够运行标准的容器镜像，它就可以通过实现CRI接入到Kubernetes项目当中。Kubernetes是通过kubelet跟CRI进行通信。

- [OCI（Open Container Initiative）](https://opencontainers.org/)：具体的容器运行时，比如Docker项目，则一般通过OCI这个容器运行时规范同底层的Linux操作系统进行交互，即：把CRI请求翻译成对Linux操作系统的调用（操作Linux Namespace和Cgroups等）。

  > CRI是k8s对外暴露的抽象容器接口，OCI是开发容器倡议，最终CRI接口会调用符合OCI的系统内核接口
  >
  > 容器生态三层抽象:
  >
  > > Orchestration API -> Container API -> Kernel API
  >
  > - Orchestration API: kubernetes API标准就是这层的标准,无可非议
  > - Container API: 标准就是CRI
  > - Kernel API: 标准就是OCI

- `CNI（Container Networking Interface）`：调用网络插件做网络通信。

- `CSI（Container Storage Interface）`：调用存储插件配置持久化存储。

- kubelet还通过gRPC协议同一个叫作Device Plugin的插件进行交互。

> 以上几种接口都是kubelet所要支持的功能，所以计算节点上最核心的组件就是kubelet。