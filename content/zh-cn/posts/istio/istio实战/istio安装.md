---
title: "istio安装"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"云原生"
]
tags : [
"云原生",
"istio"
]
series : [
"istio实战"
]
images : [
"images/center.png"
]
---

istio安装

## 1. 下载 Istio

这里推荐直接下载tar.gz安装包，不推荐使用官网上的那个安装脚本（慢得一逼）。另外推荐下载istio-1.5.10，不推荐下载1.7.0。

解压：

```shell
tar -xzvf Istio-1.5.10-linux.tar.gz
```

拷贝`istioctl`到`/usr/local/bin/`

```shell
cd Istio-1.5.10/
cp bin/istioctl /usr/local/bin/
```

查看版本：

```shell
istioctl version
```

## 2. 安装istio

基于`demo`的配置安装`istio`（除了`demo`，还有`default`等等，具体配置见`istio-1.5.10/install/kubernetes/operator/profiles`）

```shell
istioctl manifest apply --set profile=demo
```

查看svc：`kubectl get svc -n istio-system`

![img](https://picgo.6and.ltd/img/img_5f4c6db833900-20210621140453436.png)

查看pod：`kubectl get pods -n istio-system`

![img](https://picgo.6and.ltd/img/img_5f4c6dd8cc2e9-20210621140456027.png)

这里需要注意，如果使用的是`1.5.10`以后的高版本，安装命令应该是：`istioctl manifest install --set profile=demo`

并且，最新版本`1.7.0`不再默认安装`grafana ``kiali ``zipkin`等等组件。

当使用 `kubectl apply` 来部署应用时，如果 pod 启动在标有 `istio-injection=enabled` 的命名空间中，那么，[Istio sidecar 注入器](https://istio.io/latest/zh/docs/setup/additional-setup/sidecar-injection/#automatic-sidecar-injection)将自动注入 Envoy 容器到应用的 pod 中：

```shell
kubectl label namespace <namespace> Istio-injection=enabled
```

## 3.卸载

卸载程序将删除 RBAC 权限、istio-system 命名空间和所有相关资源。可以忽略那些不存在的资源的报错，因为它们可能已经被删除掉了。

```shell
istioctl manifest generate --set profile=demo | kubectl delete -f -
```

## 参考

[istio官网: 开始](https://istio.io/latest/zh/docs/setup/getting-started/)
