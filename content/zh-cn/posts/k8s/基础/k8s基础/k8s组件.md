---
title: "k8s组件"
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



<img src="http://picgo.6and.ltd/img/components-of-kubernetes.svg" alt="Kubernetes 组件" style="zoom: 60%;" />

## Master（控制平面）组件

### 1⃣️ [kube-apiserver](https://feisky.gitbooks.io/kubernetes/content/components/apiserver.html)

- 提供集群管理的 [REST API](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/) 接口，包括认证授权、数据校验以及集群状态变更等
- 提供其他模块之间的数据交互和通信的枢纽（其他模块通过 API Server 查询或修改数据，只有 API Server 才直接操作 etcd）

### 2⃣️ [etcd](https://feisky.gitbooks.io/kubernetes/content/components/etcd.html)

etcd 是 CoreOS 基于 Raft 开发的分布式 key-value 存储，可用于服务发现、共享配置以及一致性保障（如数据库选主、分布式锁等）。

 主要功能🔧包括：

- 基本的 key-value 存储
- 监听机制
- key 的过期及续约机制，用于监控和服务发现
- 原子 CAS 和 CAD，用于分布式锁和 leader 选举

 Etcd，Zookeeper，Consul 比较 🆚：

- Etcd 和 Zookeeper 提供的能力非常相似，都是通用的一致性元信息存储，都提供 watch 机制用于变更通知和分发，也都被分布式系统用来作为共享信息存储，在软件生态中所处的位置也几乎是一样的，可以互相替代的。二者除了实现细节，语言，一致性协议上的区别，**最大的区别在周边生态圈**。Zookeeper 是 apache 下的，用 java 写的，提供 rpc 接口，最早从 hadoop 项目中孵化出来，在分布式系统中得到广泛使用（hadoop, solr, kafka, mesos 等）。Etcd 是 coreos 公司旗下的开源产品，比较新，以其简单好用的 rest 接口以及活跃的社区俘获了一批用户，在新的一些集群中得到使用（比如 kubernetes）。虽然 v3 为了性能也改成二进制 rpc 接口了，但其易用性上比 Zookeeper 还是好一些。
- 而 Consul 的目标则更为具体一些，Etcd 和 Zookeeper 提供的是分布式一致性存储能力，具体的业务场景需要用户自己实现，比如服务发现，比如配置变更。而 Consul 则以服务发现和配置变更为主要目标，同时附带了 kv 存储。

### 3⃣️ [kube-scheduler](https://kubernetes.io/zh/docs/concepts/overview/components/)

kube-scheduler 负责分配调度 Pod 到集群内的节点上，它监听 kube-apiserver，查询还未分配 Node 的 Pod，然后根据调度策略为这些 Pod 分配节点（更新 Pod 的 `NodeName` 字段）。

调度器需要充分考虑🤔诸多的因素：

- 公平调度
- 资源高效利用
- QoS
- affinity 和 anti-affinity
- 数据本地化（data locality）
- 内部负载干扰（inter-workload interference）
- deadlines

### 4⃣️ [kube-controller-manager](https://feisky.gitbooks.io/kubernetes/content/components/controller-manager.html#kube-controller-manager)

kube-controller-manager 由一系列的控制器组成，这些控制器可以划分为三组

1. 必须启动的控制器

   - EndpointController: 填充端点(Endpoints)对象(即加入 Service 与 Pod)
   - ReplicationController
   - PodGCController
   - ResourceQuotaController
   - NamespaceController
   - ServiceAccountController: 为新的命名空间创建默认帐户和 API 访问令牌
   - GarbageCollectorController
   - DaemonSetController
   - JobController：监测代表一次性任务的 Job 对象，然后创建 Pods 来运行这些任务直至完成
   - DeploymentController
   - ReplicaSetController
   - HPAController
   - DisruptionController
   - StatefulSetController
   - CronJobController
   - CSRSigningController
   - CSRApprovingController
   - TTLController
2. 默认启动的可选控制器，可通过选项设置是否开启

   - TokenController: 为新的命名空间创建 API 访问令牌
   - NodeController: 负责在节点出现故障时进行通知和响应
   - ServiceController
   - RouteController
   - PVBinderController
   - AttachDetachController
3. 默认禁止的可选控制器，可通过选项设置是否开启

   - BootstrapSignerController
   - TokenCleanerController

### 5⃣️ [cloud-controller-manager](https://feisky.gitbooks.io/kubernetes/content/components/controller-manager.html#kube-controller-manager)

cloud-controller-manager 在 Kubernetes 启用 Cloud Provider 的时候才需要，用来配合云服务提供商的控制，也包括一系列的控制器

- CloudNodeController: 用于在节点终止响应后检查云提供商以确定节点是否已被删除
- RouteController: 用于在底层云基础架构中设置路由
- ServiceController: 用于创建、更新和删除云提供商负载均衡器

## Node 组件

### 1⃣️ [Kubelet](https://feisky.gitbooks.io/kubernetes/content/components/kubelet.html)

每个Node节点上都运行一个 Kubelet 服务进程，默认监听 10250 端口，接收并执行 Master 发来的指令，管理 Pod 及 Pod 中的容器。每个 Kubelet 进程会在 API Server 上注册所在Node节点的信息，定期向 Master 节点汇报该节点的资源使用情况，并通过 cAdvisor 监控节点和容器的资源。`管理节点`  `管理POD`  `管理容器`  `监控节点和容器`

### 2⃣️ [kube-proxy](https://feisky.gitbooks.io/kubernetes/content/components/kube-proxy.html)

每台机器上都运行一个 kube-proxy 服务，它监听 API server 中 service 和 endpoint 的变化情况，并通过 userspace、iptables、ipvs 或 winuserspace 等 proxier 来为服务配置负载均衡（仅支持 TCP 和 UDP）。

kube-proxy 可以直接运行在物理机上，也可以以 static pod 或者 daemonset 的方式运行。

> #### 静态 Pod
>
> 除了 DaemonSet，还可以使用静态 Pod 来在每台机器上运行指定的 Pod，这需要 kubelet 在启动的时候指定 manifest 目录：
>
> ```sh
> kubelet --pod-manifest-path=/etc/kubernetes/manifests
> ```
>
> 然后将所需要的 Pod 定义文件放到指定的 manifest 目录中。
>
> 注意：静态 Pod 不能通过 API Server 来删除，但可以通过删除 manifest 文件来自动删除对应的 Pod。

### 3⃣️ [容器运行时（Container Runtime）](https://kubernetes.io/zh/docs/concepts/overview/components/)

容器运行环境是负责运行容器的软件。

Kubernetes 支持多个容器运行环境: [Docker](https://kubernetes.io/zh/docs/reference/kubectl/docker-cli-to-kubectl/)、 [containerd](https://containerd.io/docs/)、[CRI-O](https://cri-o.io/#what-is-cri-o) 以及任何实现 [Kubernetes CRI (容器运行环境接口)](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md)。

## 插件（Addons）

### 1⃣️ [DNS](https://feisky.gitbooks.io/kubernetes/content/components/kube-dns.html)

DNS 是 Kubernetes 的核心功能之一，通过 kube-dns 或 CoreDNS 作为集群的必备扩展来提供命名服务。

从 v1.11 开始可以使用 [CoreDNS](https://coredns.io/) 来提供命名服务，并从 v1.13 开始成为默认 DNS 服务。CoreDNS 的特点是效率更高，资源占用率更小，推荐使用 CoreDNS 替代 kube-dns 为集群提供 DNS 服务。

### 2⃣️ [Dashboard](https://kubernetes.io/zh/docs/tasks/access-application-cluster/web-ui-dashboard/)

<img src="http://picgo.6and.ltd/img/ui-dashboard.png" alt="Kubernetes Dashboard UI" style="zoom: 33%;" />

### 3⃣️ [容器资源监控](https://kubernetes.io/zh/docs/tasks/debug-application-cluster/resource-usage-monitoring/)

监控系统用于采集node和pod的监控数据

- metric-server 核心指标监控
- prometheus 自定义指标监控，提供丰富功能
- heapster+influxdb+grafana 旧核心指标监控方案，现已废弃

### 4⃣️ [集群层面日志](https://kubernetes.io/zh/docs/concepts/cluster-administration/logging/)

日志采集系统，用于收集容器的业务数据,实现日志的采集，存储和展示，由EFK实现

- Fluentd 日志采集
- ElasticSearch 日志存储+检索
- Kiabana 数据展示


参考链接

https://feisky.gitbooks.io/kubernetes/content/components/components.html