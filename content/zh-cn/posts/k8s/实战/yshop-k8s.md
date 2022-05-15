+++
title = "Yshop K8s"
date = 2022-05-15T14:46:38+08:00
featured = false
draft = false
comment = true
toc = true
reward = true
pinned = false
categories= [
"云原生"
]
tags = [
"云原生",
"k8s"
]
series = [
"k8s实战"
]
images = []

+++



<!--more-->

## nacos部署

x x

## seata部署

⚠️ seata latest镜像有bug（`docker.io/seataio/seata-server:latest`,  `2de39e877c79`）无法注册到nacos，v1.4.2可正常注册

```yaml
apiVersion: v1
kind: Service
metadata:
  name: seata-server
  namespace: yshop-cloud
  labels:
    k8s-app: seata-server
spec:
  type: NodePort
  ports:
    - port: 8091
      nodePort: 30091
      protocol: TCP
      name: http
  selector:
    k8s-app: seata-server

---


apiVersion: apps/v1
kind: Deployment
metadata:
  name: seata-server
  namespace: yshop-cloud
  labels:
    k8s-app: seata-server
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: seata-server
  template:
    metadata:
      labels:
        k8s-app: seata-server
    spec:
      containers:
        - name: seata-server
          image: docker.io/seataio/seata-server:1.4.2
          imagePullPolicy: IfNotPresent
          env:
            - name: SEATA_CONFIG_NAME
              value: file:/root/seata-config/registry
          ports:
            - name: http
              containerPort: 8091
              protocol: TCP
          volumeMounts:
            - name: seata-config
              mountPath: /root/seata-config
      volumes:
        - name: seata-config
          configMap:
            name: seata-server-config

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: seata-server-config
data:
  registry.conf: |
    registry {
        type = "nacos"
        nacos {
          application = "seata-server"
          serverAddr = "nacos-0.nacos-headless.yshop-cloud.svc.cluster.local:8848,nacos-1.nacos-headless.yshop-cloud.svc.cluster.local:8848,nacos-2.nacos-headless.yshop-cloud.svc.cluster.local:8848"
          group = "YSHOP_CLOUD"
        	namespace = "e9234393-1ce4-4195-9937-5328ef05872b"
        }
    }
    config {
      type = "nacos"
      nacos {
        serverAddr = "nacos-0.nacos-headless.yshop-cloud.svc.cluster.local:8848,nacos-1.nacos-headless.yshop-cloud.svc.cluster.local:8848,nacos-2.nacos-headless.yshop-cloud.svc.cluster.local:8848"
        group = "YSHOP_CLOUD"
        namespace = "e9234393-1ce4-4195-9937-5328ef05872b"
      }
    }
```

