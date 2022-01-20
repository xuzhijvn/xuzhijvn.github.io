---
title: "istio启用策略检查功能"
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

]
---

istio启用策略检查功能

不得不说，istio的官方文档真的很垃圾，操作手册没有随着版本同步更新不说，启用策略检查功能依照[官方做法](https://istio.io/latest/zh/docs/tasks/policy-enforcement/enabling-policy/)也不能生效。现先以`istio-1.5.10`为例总结启用策略检查功能的方法如下：

1. 默认安装istio之后`disablePolicyChecks=true`

```shell
[root@k8s-master Istio-1.5.10]# kubectl -n Istio-system get cm Istio -o jsonpath="{@.data.mesh}" | grep disablePolicyChecks
disablePolicyChecks: true
```

2. 编辑 `istio configmap` 以启用策略检查

```shell
[root@k8s-master Istio-1.5.10]# kubectl -n Istio-system get cm Istio -o jsonpath="{@.data.mesh}" | sed -e "s/disablePolicyChecks: true/disablePolicyChecks: false/" > /tmp/mesh.yaml
[root@k8s-master Istio-1.5.10]# kubectl -n Istio-system create cm Istio -o yaml --dry-run --from-file=mesh=/tmp/mesh.yaml | kubectl replace -f -
W0902 17:00:37.208604   16538 helpers.go:535] --dry-run is deprecated and can be replaced with --dry-run=client.
configmap/Istio replaced
[root@k8s-master Istio-1.5.10]# kubectl -n Istio-system get cm Istio -o jsonpath="{@.data.mesh}" | grep disablePolicyChecks
disablePolicyChecks: false
```

3. 删除为修补 `istio configmap` 而创建的临时文件

```shell
rm /tmp/mesh.yaml
```

4. 总结

虽然此方法可以设置`disablePolicyChecks=false`，但是还是无法启用`istio速率限制（Rate Limits）`，具体原因不详（应该是该版本的bug），请使用`istio-1.4.10`

5. 参考：

[第二十一部分 Istio策略检查](https://blog.csdn.net/u013538795/article/details/90241062)
