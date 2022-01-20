---
title: "部署sentinel-dashboard"
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


部署sentinel-dashboard

## 1. 构建镜像

```dockerfile
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ADD sentinel-dashboard-1.8.0.jar sentinel-dashboard.jar
CMD java ${JAVA_OPTS} -jar sentinel-dashboard.jar
EXPOSE 8718
```

## 2. 创建headless service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: sentinel-headless
  labels:
    app: sentinel
  annotations:
    service.alpha.kubernetes.io/tolerate-unready-endpoints: "true"
spec:
  ports:
    - port: 8718
      name: server
      targetPort: 8718
  clusterIP: None
  selector:
    app: sentinel
```

## 3. 创建StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sentinel
spec:
  serviceName: sentinel
  replicas: 1
  template:
    metadata:
      labels:
        app: sentinel
      annotations:
        pod.alpha.kubernetes.io/initialized: "true"
    spec:
      containers:
        - name: sentinel
          imagePullPolicy: IfNotPresent
          image: ccr.ccs.tencentyun.com/xuzhijun/sentinel-dashboard:latest
          resources:
            requests:
              memory: "512Mi"
              cpu: "200m"
          ports:
            - containerPort: 8719
              name: client
          env:
            - name: TZ
              value: Asia/Shanghai
            - name: JAVA_OPTS
              value: "-Dserver.port=8718 -Dcsp.sentinel.dashboard.server=localhost:8718 -Dproject.name=sentinel-dashboard -Djava.security.egd=file:/dev/./urandom -Dcsp.sentinel.api.port=8719"
      imagePullSecrets:
      - name: registry-secret-tencent
  selector:
    matchLabels:
      app: sentinel
```

## 4. 通过Ingress暴露服务

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress-yshopcloud
  annotations:
    kubernetes.io/ingress.class: "nginx"
    # 开启use-regex，启用path的正则匹配
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  rules:
    # 定义域名
    - host: sentinel.6and.ltd
      http:
        paths:
        # 不同path转发到不同端口
          - path: /
            backend:
              serviceName: sentinel-headless
              servicePort: 8718
```

 

