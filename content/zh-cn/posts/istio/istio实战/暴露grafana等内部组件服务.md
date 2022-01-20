---
title: "暴露grafana等内部组件服务"
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


暴露grafana等内部组件服务

在安装完成istio后，默认状态下，集群外用户不能直接访问istio集群内的grafana等管理、监控服务。

有两种方法可以将集群内服务开放出来。一种是使用port-forward方式将本地端口流量转发到pod端口，实现集群内服务的访问；另一种方式是采用istio gateway方式，将集群内服务暴露到外网。

第一种方式（以暴露Prometheus为例，[官方教程](https://istio.io/latest/zh/docs/tasks/observability/metrics/querying-metrics/)也是这种方式，这种方式极其反人类，推荐使用下面的第二种方式）：

```shell
kubectl -n Istio-system port-forward $(kubectl -n Istio-system get pod -l app=prometheus -o jsonpath='{.items[0].metadata.name}') 9090:9090 &
```

第二种方式需要将集群的默认网关服务ingressgateway的网络模式设置为LB/nodeport模式，作为代理实现对外服务。

## 1. 设置ingress gateway的工作模式

安装istio的时候默认就是LB的

## 2. 验证ingress gateway的网络模式

```shell
[root@k8s-master ~]# kubectl get svc -n Istio-system
NAME                        TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                                                                                      AGE
grafana                     ClusterIP      10.101.122.19    <none>        3000/TCP                                                                                                                                     11h
Istio-egressgateway         ClusterIP      10.106.2.83      <none>        80/TCP,443/TCP,15443/TCP                                                                                                                     11h
Istio-ingressgateway        LoadBalancer   10.110.238.41    <pending>     15020:31598/TCP,80:30299/TCP,443:31413/TCP,15029:30249/TCP,15030:32499/TCP,15031:31399/TCP,15032:32373/TCP,31400:30156/TCP,15443:30319/TCP   11h
Istio-pilot                 ClusterIP      10.103.43.172    <none>        15010/TCP,15011/TCP,15012/TCP,8080/TCP,15014/TCP,443/TCP                                                                                     11h
istiod                      ClusterIP      10.105.251.21    <none>        15012/TCP,443/TCP                                                                                                                            11h
jaeger-agent                ClusterIP      None             <none>        5775/UDP,6831/UDP,6832/UDP                                                                                                                   11h
jaeger-collector            ClusterIP      10.99.197.254    <none>        14267/TCP,14268/TCP,14250/TCP                                                                                                                11h
jaeger-collector-headless   ClusterIP      None             <none>        14250/TCP                                                                                                                                    11h
jaeger-query                ClusterIP      10.110.152.152   <none>        16686/TCP                                                                                                                                    11h
kiali                       ClusterIP      10.100.252.210   <none>        20001/TCP                                                                                                                                    11h
prometheus                  ClusterIP      10.108.175.66    <none>        9090/TCP                                                                                                                                     11h
tracing                     ClusterIP      10.102.204.150   <none>        80/TCP                                                                                                                                       11h
zipkin                      ClusterIP      10.104.104.190   <none>        9411/TCP                                                                                                                                     11h
```

## 3. 查看作为边界代理的ingress-gateway的端口映射情况

```shell
[root@k8s-master ~]# kubectl describe svc Istio-ingressgateway -n Istio-system
Name:                     Istio-ingressgateway
Namespace:                Istio-system
Labels:                   app=Istio-ingressgateway
                          Istio=ingressgateway
                          operator.Istio.io/component=IngressGateways
                          operator.Istio.io/managed=Reconcile
                          operator.Istio.io/version=1.5.10
                          release=Istio
Annotations:              Selector:  app=Istio-ingressgateway,Istio=ingressgateway
Type:                     LoadBalancer
IP:                       10.110.238.41
Port:                     status-port  15020/TCP
TargetPort:               15020/TCP
NodePort:                 status-port  31598/TCP
Endpoints:                10.109.131.26:15020
Port:                     http2  80/TCP
TargetPort:               80/TCP
NodePort:                 http2  30299/TCP
Endpoints:                10.109.131.26:80
Port:                     https  443/TCP
TargetPort:               443/TCP
NodePort:                 https  31413/TCP
Endpoints:                10.109.131.26:443
Port:                     kiali  15029/TCP
TargetPort:               15029/TCP
NodePort:                 kiali  30249/TCP
Endpoints:                10.109.131.26:15029
Port:                     prometheus  15030/TCP
TargetPort:               15030/TCP
NodePort:                 prometheus  32499/TCP
Endpoints:                10.109.131.26:15030
Port:                     grafana  15031/TCP
TargetPort:               15031/TCP
NodePort:                 grafana  31399/TCP
Endpoints:                10.109.131.26:15031
Port:                     tracing  15032/TCP
TargetPort:               15032/TCP
NodePort:                 tracing  32373/TCP
Endpoints:                10.109.131.26:15032
Port:                     tcp  31400/TCP
TargetPort:               31400/TCP
NodePort:                 tcp  30156/TCP
Endpoints:                10.109.131.26:31400
Port:                     tls  15443/TCP
TargetPort:               15443/TCP
NodePort:                 tls  30319/TCP
Endpoints:                10.109.131.26:15443
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

上图所示，ingressgateway创建时，自动预设了一些端口映射，其中https-grafana的15301端口映射到node的31399端口,我们将15031端口关联到grafana上，集群外就用户通过访问网关所在机器的31399端口访问到grafana服务。

## 4. gateway方式暴露集群内服务

需要创建服务的gateway和virtual service资源如下：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: grafana-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 15031
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: grafana-vs
spec:
  hosts:
  - "*"
  gateways:
  - grafana-gateway
  http:
  - route:
    - destination:
        host: grafana
        port:
          number: 3000
```

执行：

```shell
kubectl apply -f grafana-gateway.yaml -n Istio-system
```

特别需要注意：后面的`namespace`是`istio-system`，因为grafana等内部组件都是这个namespace的。

## 5. 测试grafana的连通性

```shell
http://<公网IP>:31399
```

## 参考

[istio中的grafana等内部组件服务开放给集群外用户访问](https://blog.csdn.net/scwang18/article/details/100833886)
