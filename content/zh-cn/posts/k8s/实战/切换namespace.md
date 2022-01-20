---
title: "切换namespace"
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

]
---


切换namespace

## 0. 背景

k8s如何切换namespace？？而不必每次执行命令的时候在后面指定namespace？k8s并没有直接提供切换namespace的命令。不过可以通过：

- 切换context达到切换namespace的目的，这需要提前创建好context和namespace的绑定关系（如：`kubectl config set-context test --namespace=test`）,然后使用 `kubectl config use-context test `切换context，从而间接的达到切换namespace的目的。这方法属实是太别扭了，强迫症患者都不喜欢。
- 或者使用`kubectx`工具，其本质是动态的修改context和namespace的绑定关系。使用形如：`kubens test` 即可切换。这样才是切换namespace的正确打开方式，优雅多了。

## 参考资料：

[Kubernetes命名空间](https://www.cnblogs.com/cocowool/p/kubernetes_namespace.html)

[一条命令解决Kubernetes更改默认的namespace](https://www.jianshu.com/p/e3fd90603229)

[Kubernetes 切换context和namespace](https://blog.csdn.net/qq_19734597/article/details/98175134)

[k8s集群namespace和context使用](https://blog.csdn.net/skh2015java/article/details/108409458)

[ahmetb / kubectx](https://github.com/ahmetb/kubectx)

[提高您的kubectl生产力（第三部分）：集群上下文切换、使用别名减少输入和插件扩展](https://tonybai.com/2019/08/31/kubectl-productivity-part3/)
