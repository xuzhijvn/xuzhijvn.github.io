---
title: "istio实现灰度"
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

istio实现灰度

![Bookinfo Application without Istio](http://picgo.6and.ltd/img/noistio.svg)

# Gateway

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
```

# VirtualService

是在 Istio 服务网格内对服务的请求如何进行路由控制？[`VirtualService`](https://preliminary.istio.io/zh/docs/reference/config/istio.networking.v1alpha3/#virtualservice) 中就包含了这方面的定义。例如一个 Virtual Service 可以把请求路由到不同版本，甚至是可以路由到一个完全不同于请求要求的服务上去。路由可以用很多条件进行判断，例如请求的源和目的地、HTTP 路径和 Header 以及各个服务版本的权重等。

路由规则对应着一或多个用 `VirtualService` 配置指定的请求目的主机。这些主机可以是也可以不是实际的目标负载，甚至可以不是同一网格内可路由的服务。例如要给到 `reviews` 服务的请求定义路由规则，可以使用内部的名称 `reviews`，也可以用域名 `bookinfo.com`，`VirtualService` 可以定义这样的 `host` 字段：

```
hosts:
  - reviews
  - bookinfo.com复制代码
```

`host` 字段用显示或者隐式的方式定义了一或多个完全限定名（FQDN）。上面的 `reviews`，会隐式的扩展成为特定的 FQDN，例如在 Kubernetes 环境中，全名会从 `VirtualService` 所在的集群和命名空间中继承而来（比如说 `reviews.default.svc.cluster.local`）。

```yaml
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
```

所有的流量都去到reviews:v1

<img src="http://picgo.6and.ltd/img/image-20210601124539407.png" alt="image-20210601124539407" style="zoom:40%;" />

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
```

来自名为 Jason 的用户的所有流量将被路由到服务 `reviews:v2`。

<img src="http://picgo.6and.ltd/img/image-20210601123957608-20210601124059926.png" style="zoom:40%;" />

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
```

把 50% 的流量从 `reviews:v1` 转移到 `reviews:v3`：

<img src="http://picgo.6and.ltd/img/image-20210601124730191.png" alt="image-20210601124730191" style="zoom:40%;" />

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 50
    - destination:
        host: reviews
        subset: v3
      weight: 50
```

给对 `reviews` 服务的调用增加一个半秒的请求超时：

<img src="http://picgo.6and.ltd/img/image-20210601124329947.png" alt="image-20210601124329947" style="zoom:40%;" />

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v2
    timeout: 0.5s
```

# DestinationRule

在请求被 `VirtualService` 路由之后，[`DestinationRule`](https://preliminary.istio.io/zh/docs/reference/config/istio.networking.v1alpha3/#destinationrule) 配置的一系列策略就生效了。这些策略⚠️**由服务属主编写**⚠️，包含断路器、负载均衡以及 TLS 等的配置内容。

`DestinationRule` 还定义了对应目标主机的可路由 `subset`（例如有命名的版本）。`VirtualService` 在向特定服务版本发送请求时会用到这些子集。

下面是 `reviews` 服务的 `DestinationRule` 配置策略以及子集：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  trafficPolicy:
    loadBalancer:
      simple: RANDOM
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
  - name: v3
    labels:
      version: v3复制代码
```

注意在单个 `DestinationRule` 配置中可以包含多条策略（比如 default 和 v2）。

⚠️到了DestinationRule层面，是配置流量去到指定版本的策略，随机策略、轮训策略等等（注意指定版本是可以多副本的）

可以用一系列的标准，例如连接数和请求数限制来定义简单的断路器。

例如下面的 `DestinationRule` 给 `reviews` 服务的 `v1` 版本设置了 100 连接的限制：

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
    trafficPolicy:
      connectionPool:
        tcp:
          maxConnections: 100
```

