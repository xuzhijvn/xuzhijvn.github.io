---
title: "ingress实现灰度"
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



### Nginx-ingress 架构和原理

迅速回顾一下 `Nginx-ingress` 的架构和实现原理：

![img](http://cdn.tkaid.com/img/20200727162517.png)

`Nginx-ingress` 通过前置的 `Loadbalancer` 类型的 `Service` 接收集群流量，将流量转发至 `Nginx-ingress` Pod 内并对配置的策略进行检查，再转发至目标 `Service`，最终将流量转发至业务容器。

传统的 `Nginx` 需要我们配置 `conf` 文件策略。但 `Nginx-ingress` 通过实现 `Nginx-ingress-Controller` 将原生 `conf` 配置文件和 `yaml` 配置文件进行了转化，当我们配置 `yaml` 文件的策略后，`Nginx-ingress-Controller` 将对其进行转化，并且动态更新策略，动态 Reload `Nginx Pod`，实现自动管理。

那么 `Nginx-ingress-Controller` 如何能够动态感知集群的策略变化呢？方法有很多种，可以通过 webhook admission 拦截器，也可以通过 ServiceAccount 与 Kubernetes Api 进行交互，动态获取。`Nginx-ingress-Controller` 使用后者来实现。所以在部署 `Nginx-ingress` 我们会发现 `Deployment` 内指定了 Pod 的 ServiceAccount，以及实现了 RoleBinding ，最终达到 Pod 与 Kubernetes Api 交互的目标。



# dev

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: dev
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: wangweicoding-docker.pkg.coding.net/nginx-ingress-gray/docker/nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: dev
spec:
  ports:
  - name: tcp-80-80
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
     app: nginx
  sessionAffinity: None
  type: NodePort
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx  # nginx=nginx-ingress| qcloud=CLB ingress
    ## kubernetes.io/ingress.subnetId: subnet-xxxxxxxx   # if qcloud, should give subnet
  name: my-ingress
  namespace: dev
spec:
  rules:
  - host: nginx-ingress.coding.dev
    http:
      paths:
      - backend:
          serviceName: nginx
          servicePort: 80
        path: /
```



# Prd

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: pro
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: pro
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: wangweicoding-docker.pkg.coding.net/nginx-ingress-gray/docker/nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: pro
spec:
  ports:
  - name: tcp-80-80
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
     app: nginx
  sessionAffinity: None
  type: NodePort
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx  # nginx=nginx-ingress| qcloud=CLB ingress
    ## kubernetes.io/ingress.subnetId: subnet-xxxxxxxx   # if qcloud, should give subnet
  name: my-ingress
  namespace: pro
spec:
  rules:
  - host: nginx-ingress.coding.pro
    http:
      paths:
      - backend:
          serviceName: nginx
          servicePort: 80
        path: /
```



# Canary

![img](https://help-assets.codehub.cn/enterprise/20200727163900.png)

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: pro
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-canary
  namespace: pro
spec:
  selector:
    matchLabels:
      app: nginx-canary
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx-canary
    spec:
      containers:
      - name: nginx
        image: wangweicoding-docker.pkg.coding.net/nginx-ingress-gray/docker/nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-canary
  namespace: pro
spec:
  ports:
  - name: tcp-80-80
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
     app: nginx-canary
  sessionAffinity: None
  type: NodePort
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx  # nginx=nginx-ingress| qcloud=CLB ingress
    ## kubernetes.io/ingress.subnetId: subnet-xxxxxxxx   # if qcloud, should give subnet
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-by-header: "location"
    nginx.ingress.kubernetes.io/canary-by-header-value: "shenzhen"
    #nginx.ingress.kubernetes.io/canary-weight: 100
  name: my-ingress
  namespace: pro
spec:
  rules:
  - host: nginx-ingress.coding.pro
    http:
      paths:
      - backend:
          serviceName: nginx-canary
          servicePort: 80
        path: /
```

