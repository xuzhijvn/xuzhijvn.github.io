---
title: "多Pod实现灰度"
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



如果您想使用 Deployment 将最新的应用程序版本发布给一部分用户（或服务器），您可以为每个版本创建一个 Deployment，此时，应用程序的新旧两个版本都可以同时获得生产上的流量。

<img src="http://picgo.6and.ltd/img/%EF%BC%8C.png" alt="，" style="zoom: 33%;" />

# 实施方案

- 部署第一个版本

  第一个版本的 Deployment 包含了 3 个Pod副本，Service 通过 label selector `app: nginx` 选择对应的 Pod，nginx 的标签为 `1.7.9`

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment
    labels:
      app: nginx
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:1.7.9
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-service
    labels:
      app: nginx
  spec:
    selector:
      app: nginx
    ports:
    - name: nginx-port
      protocol: TCP
      port: 80
      nodePort: 32600
      targetPort: 80
    type: NodePort
  ```

- 假设此时想要发布新的版本 nginx `1.8.0`，可以创建第二个 Deployment：

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment-canary
    labels:
      app: nginx
      track: canary
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: nginx
        track: canary
    template:
      metadata:
        labels:
          app: nginx
          track: canary
      spec:
        containers:
        - name: nginx
          image: nginx:1.8.0
  ```

  - 因为 Service 的LabelSelector 是 `app: nginx`，由 `nginx-deployment` 和 `nginx-deployment-canary` 创建的 Pod 都带有标签 `app: nginx`，所以，Service 的流量将会在两个 release 之间分配
  - 在新旧版本之间，流量分配的比例为两个版本副本数的比例，此处为 1:3
  - 当您确定新的版本没有问题之后，可以将 `nginx-deployment` 的镜像标签修改为新版本的镜像标签，并在完成对 `nginx-deployment` 的滚动更新之后，删除 `nginx-deployment-canary` 这个 Deployment

# 局限性

按照 Kubernetes 默认支持的这种方式进行金丝雀发布，有一定的局限性：

- 不能根据用户注册时间、地区等请求中的内容属性进行流量分配
- 同一个用户如果多次调用该 Service，有可能第一次请求到了旧版本的 Pod，第二次请求到了新版本的 Pod

# TIP

在 Kubernetes 中不能解决上述局限性的原因是：Kubernetes Service 只在 TCP 层面解决负载均衡的问题，并不对请求响应的消息内容做任何解析和识别。如果想要更完善地实现金丝雀发布，可以考虑如下三种选择：

- 业务代码编码实现
- Spring Cloud 灰度发布
- Istio 灰度发布
