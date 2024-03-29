---
title: "k8s组件间通信"
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



Kubernetes 多组件之间的通信原理：

- apiserver 负责 etcd 存储的所有操作，且只有 apiserver 才直接操作 etcd 集群
- apiserver 对内（集群中的其他组件）和对外（用户）提供统一的 REST API，其他组件均通过 apiserver 进行通信
  - controller manager、scheduler、kube-proxy 和 kubelet 等均通过 apiserver watch API 监测资源变化情况，并对资源作相应的操作
  - 所有需要更新资源状态的操作均通过 apiserver 的 REST API 进行
- apiserver 也会直接调用 kubelet API（如 logs, exec, attach 等），默认不校验 kubelet 证书，但可以通过 `--kubelet-certificate-authority` 开启（而 GKE 通过 SSH 隧道保护它们之间的通信）

比如最典型的创建 Pod 的流程：

<img src="http://cdn.tkaid.com/img/k8s-pod-process.png" alt="k8s pod" style="zoom:67%;" />

1. 通过 `CLI 或者 UI` 提交 Pod 部署请求给 Kubernetes API Server
2. `API Server` 会把这个信息写入到它的存储系统 etcd
3. `Scheduler` 会通过 API Server 的 watch 或者叫做 notification 机制得到这个信息：有一个 Pod 需要被调度
   - 调度决策
   - 向 API Server report 说：“OK！这个 Pod 需要被调度到某一个节点上。”

4. `API Server` 接收到这次操作之后，会把这次的结果再次写到 etcd 中
5. 相应节点的 `kubelet` 会watch到一个需要启动的Pod
   - kubelet调CRI配置容器的运行环境、调CNI配置网络、调CSI接口配置存储
   - 上传状态信息给API Server 
6. `API Server` 记录Pod部署状态到etcd

参考链接

https://www.qikqiak.com/k8s-book/docs/15.%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5%E4%B8%8E%E7%BB%84%E4%BB%B6.html
https://www.wumingx.com/k8s/kubernetes-introduction.html
