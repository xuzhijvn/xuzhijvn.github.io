---
title: "ingress"
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



## 1. ingress是什么

`ingress`是k8s一种资源对象，如k8s的`Deployment`、`Service`资源对象一样。它是一种集群维度暴露服务的方式，正如k8s的`ClusterIP`、`NodePort`、`LoadBalance`一样，但是ClusterIP的方式只能在集群内部访问，NodePort方式的话，测试环境使用还行，当有几十上百的服务在集群中运行时，NodePort的端口管理是灾难，LoadBalance方式受限于云平台，且通常在云平台部署ELB还需要额外的费用。ingress规则是很灵活的，可以根据不同域名、不同path转发请求到不同的service，并且支持https/http。

<img src="https://picgo.6and.ltd/img/img_5f40a321e4f83.png" alt="ingress" style="zoom:67%;" />

## 2. ingress与ingress-controller

要理解ingress，需要区分两个概念，`ingress`和`ingress-controller`：

- `ingress`：指的是k8s中的一个api对象，一般用yaml配置。作用是定义请求如何转发到service的规则，可以理解为配置模板。
- `ingress-controller`：具体实现反向代理及负载均衡的程序，对ingress定义的规则进行解析，根据配置的规则来实现请求转发。

简单来说，ingress-controller才是负责具体转发的组件，通过各种方式将它暴露在集群入口，外部对集群的请求流量会先到ingress-controller，而ingress对象是用来告诉ingress-controller该如何转发请求，比如哪些域名哪些path要转发到哪些服务等等。

### 2.1 ingress

`ingress`是一个API对象，和其他对象一样，通过yaml文件来配置。ingress通过http或https暴露集群内部service，给service提供外部URL、负载均衡、SSL/TLS能力以及基于host的方向代理。ingress要依靠ingress-controller来具体实现以上功能。前一小节的图如果用ingress来表示，大概就是如下配置：

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: abc-ingress
  annotations: 
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  tls:
  - hosts:
    - api.abc.com
    secretName: abc-tls
  rules:
  - host: api.abc.com
    http:
      paths:
      - backend:
          serviceName: apiserver
          servicePort: 80
  - host: www.abc.com
    http:
      paths:
      - path: /image/*
        backend:
          serviceName: fileserver
          servicePort: 80
  - host: www.abc.com
    http:
      paths:
      - backend:
          serviceName: feserver
          servicePort: 8080
```

### 2.2 ingress-controller

`ingress-controller`并不是k8s自带的组件，实际上ingress-controller只是一个统称，用户可以选择不同的ingress-controller实现，目前，由k8s维护的ingress-controller只有google云的`GCE`与`ingress-nginx`两个，其他还有很多第三方维护的ingress-controller，具体可以参考官方文档。但是不管哪一种ingress-controller，实现的机制都大同小异，只是在具体配置上有差异。一般来说，ingress-controller的形式都是一个pod，里面跑着daemon程序和反向代理程序。daemon负责不断监控集群的变化，根据ingress对象生成配置并应用新配置到反向代理，比如nginx-ingress就是动态生成nginx配置，动态更新upstream，并在需要的时候reload程序应用新配置。为了方便，后面的例子都以k8s官方维护的`nginx-ingress`为例。

## 3. 部署ingress对象

下面会介绍到`ingress-controller`有多种部署方式，但无论ingress-controller采用何种方式部署，`ingress`资源对象的部署都一样：

```yaml
[root@k8s-master ingress-nginx]# vi ingress-all.yaml 

apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress-all
  annotations:
    kubernetes.io/ingress.class: "nginx"
    # 开启use-regex，启用path的正则匹配
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  rules:
    # 定义域名
    - host: ceres-admin-web.6and.ltd
      http:
        paths:
        # 不同path转发到不同端口
          - path: /
            backend:
              serviceName: ceres-admin-web-svc
              servicePort: 80
          #- path: /merchant
          #  backend:
          #    serviceName: ceres-merchant-web-svc
          #    servicePort: 80
    - host: ceres-merchant-web.6and.ltd
      http:
        paths:
        # 不同path转发到不同端口
          - path: /
            backend:
              serviceName: ceres-merchant-web-svc
              servicePort: 80
    - host: ceres-admin-server.6and.ltd
      http:
        paths:
        # 不同path转发到不同端口
          - path: /
            backend:
              serviceName: ceres-admin-server-svc
              servicePort: 8764
```

## 4. 部署ingress-controller

`ingress-controller`的部署，需要考虑两个方面：

- ingress-controller是作为pod来运行的，以什么方式部署比较好
- ingress-controller解决了把如何请求路由到集群内部，那它自己怎么暴露给外部比较好

下面列举一些目前常见的部署和暴露方式，具体使用哪种方式还是得根据实际需求来考虑决定。

### 4.1 Deployment+LoadBalancer模式的Service

如果要把ingress部署在公有云，那用这种方式比较合适。用Deployment部署ingress-controller，创建一个type为LoadBalancer的service关联这组pod。大部分公有云，都会为LoadBalancer的service自动创建一个负载均衡器，通常还绑定了公网地址。只要把域名解析指向该地址，就实现了集群服务的对外暴露。

### 4.2 Deployment+NodePort模式的Service

同样用deployment模式部署ingress-controller，并创建对应的服务，但是type为NodePort。这样，ingress就会暴露在集群节点ip的特定端口上。由于nodeport暴露的端口是随机端口，一般会在前面再搭建一套负载均衡器来转发请求。该方式一般用于宿主机是相对固定的环境ip地址不变的场景。
NodePort方式暴露ingress虽然简单方便，但是NodePort多了一层NAT，在请求量级很大时可能对性能会有一定影响。

#### 4.2.1 下载部署[ingress-nginx](https://github.com/kubernetes/ingress-nginx) controller所需要的deploy.yaml文件

```shell
wget https://github.com/kubernetes/ingress-nginx/blob/master/deploy/static/provider/baremetal/deploy.yaml
```

该yaml默认使用nodePort方式部署Service，我们可以指定nodePort，而不是让k8s自己分配。

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    helm.sh/chart: ingress-nginx-2.11.1
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.34.1
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  type: NodePort
  ports:
    - name: http
      nodePort: 31080
      port: 80
      protocol: TCP
      targetPort: http
    - name: https
      nodePort: 31443
      port: 443
      protocol: TCP
      targetPort: https
```

不要试图将nodePort指定为`80`或者`443`，否则你会得到如下提示：

```shell
The Service "ingress-nginx-controller" is invalid: spec.ports[0].nodePort: Invalid value: 80: provided port is not in the valid range. The range of valid ports is 30000-32767
```

#### 4.2.2 修改镜像仓库地址

deploy.yaml里面的镜像地址是：`us.gcr.io/k8s-artifacts-prod/ingress-nginx/controller:v0.34.1@sha256:0e072dddd1f7f8fc8909a2ca6f65e76c5f0d2fcfb8be47935ae3457e8bbceb20`

该地址无法被访问，解决办法参考：[如何在国内顺畅下载被墙的 Docker 镜像？](https://mp.weixin.qq.com/s/kf0SrktAze3bT7LcIveDYw)

修改后如下图所示：

![img](https://picgo.6and.ltd/img/img_5f40b9365a422-20210621140800469.png)

#### 4.2.3 部署**ingress-nginx controller**

```
kubectl apply -f deploy.yaml
```

如果修改了yaml重新部署：

```
kubectl replace --force -f deploy.yaml
```

#### 4.2.4 检查是否部署成功

查看controller是否running

![img](https://picgo.6and.ltd/img/img_5f40b97a7c88d-20210621140805590.png)

查看service

![img](https://picgo.6and.ltd/img/img_5f40ba7ebd446-20210621140808992.png)

登陆任意节点，运行：

```shell
[root@k8s-node1 ~]# curl -i localhost:31080/healthz
HTTP/1.1 200 OK
Server: nginx/1.19.1
Date: Sat, 22 Aug 2020 06:22:15 GMT
Content-Type: text/html
Content-Length: 0
Connection: keep-alive
```

返回 200 ，说明 nginx OK。

至此，ingress-nginx controller已经部署完毕。通过`域名`+`nodePort`的方式访问相应的service，例：`http://ceres-admin-web.6and.ltd:31080`，如果你跟我一样是强迫症晚期，一定期望完美的访问方式是：`http://ceres-admin-web.6and.ltd`，下面的4.3小节将是强迫症晚期的福音。

### 4.3 DaemonSet+HostNetwork+nodeSelector

用`DaemonSet`结合nodeselector来部署ingress-controller到特定的node上，然后使用HostNetwork直接把该pod与宿主机node的网络打通，直接使用宿主机的80/433端口就能访问服务。这时，ingress-controller所在的node机器就很类似传统架构的边缘节点，比如机房入口的nginx服务器。该方式整个请求链路最简单，性能相对NodePort模式更好。缺点是由于直接利用宿主机节点的网络和端口，一个node只能部署一个ingress-controller pod。比较适合大并发的生产环境使用。

#### 4.3.1 给要部署nginx-ingress的node打上特定标签

我们需要使用daemonset部署到特定node，需要修改部分配置：先给要部署nginx-ingress的node打上特定标签，这里测试部署在`k8s-node1`这个节点。

```shell
kubectl label node k8s-node1 isIngress="true"
```

#### 4.3.2 修改deploy.yaml的deployment部分配置

![img](https://picgo.6and.ltd/img/img_5f40d9927f98b-20210621140820358.png)

删除指定的nodePort：

![img](https://picgo.6and.ltd/img/img_5f40da20d4f52-20210621140825440.png)

#### 4.3.3 重启

```shell
kubectl replace --force -f deploy.yaml
```

#### 4.3.4 检查

可以看到，nginx-controller的pod已经部署在在k8s-node1上了：

![img](https://picgo.6and.ltd/img/img_5f40dc88f1c92-20210621140832249.png)

到k8s-node1上看下本地端口：

![img](https://picgo.6and.ltd/img/img_5f40dcba3965c.png)

由于配置了hostnetwork，nginx已经在node主机本地监听80/443/8181端口。其中8181是nginx-controller默认配置的一个default backend。这样，只要访问node主机有公网IP，就可以直接映射域名来对外网暴露服务了。如果要nginx高可用的话，可以在多个node上部署，并在前面再搭建一套LVS+keepalive做负载均衡。用hostnetwork的另一个好处是，如果lvs用DR模式的话，是不支持端口映射的，这时候如果用nodeport，暴露非标准的端口，管理起来会很麻烦。

现在我们就可以优雅的使用：`http://ceres-admin-web.6and.ltd`来访问我们的服务了。

## 参考链接

[k8s ingress原理及ingress-nginx部署测试](https://segmentfault.com/a/1190000019908991)

[见异思迁：K8s 部署 Nginx Ingress Controller 之 kubernetes/ingress-nginx](https://www.cnblogs.com/dudu/p/12334613.html)

[如何在国内顺畅下载被墙的 Docker 镜像？](https://mp.weixin.qq.com/s/kf0SrktAze3bT7LcIveDYw)
