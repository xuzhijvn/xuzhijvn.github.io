---
title: "java应用从nfs加载配置文件"
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

java应用从nfs加载配置文件

## 背景

配置文件变化，无需重新构建镜像部署。

## 1. 准备nfs

1、准备好nfs服务器。参考：[nfs安装](http://106.55.152.92:30989/archives/1004) 的nfs服务端配置。
2、k8s node节点可以不启用`rpcbind`服务，但是必须安装`nfs-utils`（yum install nfs-utils），否则`nfs-client-provisioner` pod无法启动，因为`nfs-client.yaml`里面有nfs的相关配置，而这些nfs配置要生效需依赖`nfs-utils`。参考：[nfs安装](http://106.55.152.92:30989/archives/1004) 的nfs客户端配置。

## 2. 创建StorageClass对象

理论部分参考：[StorageClass](https://www.qikqiak.com/k8s-book/docs/35.StorageClass.html)

### 2.1 nfs-client.yaml

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-client-provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: quay.io/external_storage/nfs-client-provisioner:latest
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: fuseim.pri/ifs
            - name: NFS_SERVER
              value: nfs
            - name: NFS_PATH
              value: /mnt/nfs/k8s
      volumes:
        - name: nfs-client-root
          nfs:
            server: nfs
            path: /mnt/nfs/k8s
```

[v_warn]k8s node节点可以不启用`rpcbind`服务，但必须安装`nfs-utils`。[/v_warn]

### 2.2 nfs-client-sa.yaml

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner

---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["list", "watch", "create", "update", "patch"]
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["create", "delete", "get", "list", "watch", "patch", "update"]

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
```

### 2.3 nfs-client-class.yaml

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-storage
provisioner: fuseim.pri/ifs # or choose another name, must match deployment's env PROVISIONER_NAME'
```

现在我们来创建这些资源对象吧：

```shell
$ kubectl create -f nfs-client.yaml
$ kubectl create -f nfs-client-sa.yaml
$ kubectl create -f nfs-client-class.yaml
```

创建完成后查看下资源状态：

```shell
$ kubectl get pods
NAME                                             READY     STATUS             RESTARTS   AGE
...
nfs-client-provisioner-7648b664bc-7f9pk          1/1       Running            0          7h
...
$ kubectl get storageclass
NAME                 PROVISIONER      AGE
course-nfs-storage   fuseim.pri/ifs   11s
```

## 3. 创建PVC对象

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: ceres-admin-server-nfs-pvc
  annotations:
    volume.beta.kubernetes.io/storage-class: "nfs-storage"
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 32Mi
```

查看pvc：

```shell
[root@k8s-master StorageClass]# kubectl get pvc
NAME                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
ceres-admin-server-nfs-pvc   Bound    pvc-fec62c86-451e-4125-9e00-be60394f40d9   32Mi       RWX            nfs-storage    3h13m
ceres-app-server-nfs-pvc     Bound    pvc-6367fbb5-bc9c-4a75-a5a4-0a555172a6fd   32Mi       RWX            nfs-storage    53m
```

自动生成了一个关联的 PV 对象，访问模式是 RWX，回收策略是 Delete，这个 PV 对象并不是我们手动创建的吧，这是通过我们上面的 StorageClass 对象自动创建的：

```shell
[root@k8s-master StorageClass]# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                STORAGECLASS   REASON   AGE
pvc-6367fbb5-bc9c-4a75-a5a4-0a555172a6fd   32Mi       RWX            Delete           Bound    default/ceres-app-server-nfs-pvc     nfs-storage             55m
pvc-fec62c86-451e-4125-9e00-be60394f40d9   32Mi       RWX            Delete           Bound    default/ceres-admin-server-nfs-pvc   nfs-storage             3h14m
```

## 4. 拷贝配置文件到nfs共享目录

创建pvc之后就能在nfs看到已经生成相应的共享目录。

共享目录命名规则如下：

- 自动创建的 PV 以`${namespace}-${pvcName}-${pvName}`这样的命名格式创建在 NFS 服务器上的共享数据目录中
- 而当这个 PV 被回收后会以`archieved-${namespace}-${pvcName}-${pvName}`这样的命名格式存在 NFS 服务器上。

形如下图所示：
![img](https://picgo.6and.ltd/img/img_5fb0f0f9ef6a6.png)

## 5. 使用PVC

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: ceres-admin-server
  name: ceres-admin-server-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ceres-admin-server
  template:
    metadata:
      labels:
        app: ceres-admin-server
    spec:
      containers:
        - command:
            - java
            - '-jar'
            - /root/app.jar
            - '--spring.config.location=/root/config/'
            - '--spring.profiles.active=prod'
          image: anaham-docker.pkg.coding.net/cereshop/ceres/ceres-admin-server
          name: ceres-admin-server
          ports:
            - containerPort: 9000
          volumeMounts:
            - mountPath: /root/config/
              name: ceres-admin-server-nfs-pvc
          workingDir: /root
      imagePullSecrets:
        - name: coding-regcred
      volumes:
        - name: ceres-admin-server-nfs-pvc
          persistentVolumeClaim:
            claimName: ceres-admin-server-nfs-pvc
```

更新配置文件后删除pod，让k8s重新创建pod即可让修改配置生效，从而避免重新构建镜像。

