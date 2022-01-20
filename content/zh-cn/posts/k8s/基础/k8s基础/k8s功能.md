---
title: "k8s功能"
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

[comment]: <> "# 1.2.1 k8s功能"

Kubernetes提供了一个可弹性运行分布式系统的框架。Kubernetes 会满足您的扩展要求、故障转移、部署模式等。具体如下：

- `Service discovery and load balancing`，服务发现和负载均衡，通过DNS实现内部解析，service实现负载均衡
- `Storage orchestration`，存储编排，通过plungin的形式支持多种存储，如本地，nfs，ceph，公有云快存储等
- `Automated rollouts and rollbacks`，自动发布与回滚，通过匹配当前状态与目标状态一致，更新失败时可回滚
- `Automatic bin packing`，自动资源调度，可以设置pod调度的所需（requests）资源和限制资源（limits）
- `Self-healing`，内置的健康检查策略，自动发现和处理集群内的异常，更换，需重启的pod节点
- `Secret and configuration management`，密钥和配置管理，对于敏感信息如密码，账号的那个通过secret存储，应用的配置文件通过configmap存储，避免将配置文件固定在镜像中，增加容器编排的灵活性
- `Batch execution`，批处理执行，通过job和cronjob提供单次批处理任务和循环计划任务功能的实现
- `Horizontal scaling`，横向扩展功能，包含有HPA和AS，即应用的基于CPU利用率的弹性伸缩和基于平台级的弹性伸缩，如自动增加node和删除nodes节点。

> 使用kubectl autoscale命令来创建一个 HPA 对象：
>
> ```shell
> $ kubectl autoscale deployment wordpress --namespace kube-example --cpu-percent=20 --min=3 --max=6
> horizontalpodautoscaler.autoscaling/hpa-demo autoscaled
> $ kubectl get hpa -n kube-example
> NAME        REFERENCE              TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
> wordpress   Deployment/wordpress   <unknown>/20%   3         6         0          13s
> ```
>
> 此命令创建了一个关联资源 wordpress 的 HPA，最小的 Pod 副本数为3，最大为6。HPA 会根据设定的 cpu 使用率（20%）动态的增加或者减少 Pod 数量。

