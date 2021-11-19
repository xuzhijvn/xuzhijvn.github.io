---
title: "k8s部署应用(前端静态)"
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
"k8s实战"
]
images : [
"images/center.png"
]
---

k8s部署应用(前端静态)

## 0. 准备条件

1. 部署好了k8s集群，部署可以参考[Kubernetes: 从零搭建K8S](http://106.55.152.92:30989/archives/834)

| 名称   | 数量 | IP          | 备注                                                         |
| ------ | ---- | ----------- | ------------------------------------------------------------ |
| master | 1    | 172.17.0.14 | 操作系统: Linux(centos7, 其它操作系统也可, 安装过程类似, 可参考官方文档) 机器配置: 4C8G |
| node1  | 1    | 172.18.0.7  | 同上                                                         |
| node2  | 1    | 172.19.0.5  | 同上                                                         |

2. 应用已经容器化，并上传到了远程仓库，笔者是腾讯云容器仓库：
   ![img](https://picgo.6and.ltd/img/img_5efcb24401ad0-20210621141634123.png)

3. 理解k8s基础概念，可以参考[Kubernetes: 基础概念介绍](http://106.55.152.92:30989/archives/874)

## 1. 控制器管理Pod

### 1.1 生成`deployment`配置文件

```shell
kubectl create deployment yshop-h5 --image=ccr.ccs.tencentyun.com/yshop/h5 --dry-run -o yaml > yshop-h5.yaml
```

这个时候k8s还不能拉取镜像，需要生成拉取镜像的密钥。

### 1.2 生成拉取镜像的密钥

```shell
kubectl create secret docker-registry registry-secret-tencent --docker-server=ccr.ccs.tencentyun.com --docker-username=腾讯云账户ID --docker-password=腾讯云容器仓库密码
```

--docker-server: 仓库地址
--docker-username: 仓库登陆账号
--docker-password: 仓库登陆密码
--docker-email: 邮件地址(选填)
-n 命名空间(选填)

可以运行：`kubectl get secret registry-secret-tencent --output=yaml` 查看生成的密钥

### 1.3 配置密钥

```shell
vi yshop-h5.yaml
```

![img](https://picgo.6and.ltd/img/img_5efcb55bacecd-20210621141721477.png)

### 1.4 运行`deployment`

```shell
kubectl apply -f yshop-h5.yaml
```

查看启动的pods：`kubectl get pods`

![img](https://picgo.6and.ltd/img/img_5efcb676f200d-20210621141730179.png)

查看启动日志：`kubectl logs yshop-h5-cd4dc8c5b-562g5`

 

## 2. 暴露应用

### 2.1 生成`service`配置文件

```shell
kubectl expose deployment yshop-h5 --port=80 --target-port=80 --type=NodePort -o yaml --dry-run > yshop-h5-svc.yaml
```

yshop-h5 指定名称

--port 指定集群内部访问的端口

--target-port 指定容器内跑服务的端口

--type=NodePort 指定类型 集群外部访问

### 2.2 运行`service`

```shell
kubectl apply -f yshop-h5-svc.yaml
```

查看pods和svc：`kubectl get pods,svc`

![img](https://picgo.6and.ltd/img/img_5efcb91e43add-20210621141742027.png)

查看pods分布的节点： `kubectl get pods -o wide`

![img](https://picgo.6and.ltd/img/img_5efcb8975f615-20210621141745982.png)

## 3. 访问应用

### 3.1 外部访问

```shell
http://{master的公网ip/node1的公网ip/node2的公网ip}:32585
```

### 3.2 内部访问

通过service ip:

```shell
curl http://10.108.253.217
```

![img](https://picgo.6and.ltd/img/img_5efcbb070be4f.png)

通过节点ip:

```shell
 curl http://{master的内网ip/node1的内网ip/node2的内网ip}:32585
```

![img](https://picgo.6and.ltd/img/img_5efcbb9ab0aef.png)

 