pvc不会主动回收问题

使用k8s部署nacos的时候发现，通过volumeClaimTemplates配置的pvc不会随着使用`kubectl delete -f xxx.yaml` 删除StatefulSet的时候一同被删除。`nacos-pvc-nfs.yaml`文件如下所示：

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nacos-headless
  labels:
    app: nacos
  annotations:
    service.alpha.kubernetes.io/tolerate-unready-endpoints: "true"
spec:
  ports:
    - port: 8848
      name: server
      targetPort: 8848
    - port: 7848
      name: rpc
      targetPort: 7848
  clusterIP: None
  selector:
    app: nacos
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nacos-cm
data:
  mysql.db.name: "nacos_devtest"
  mysql.port: "3306"
  mysql.user: "nacos"
  mysql.password: "nacos"
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nacos
spec:
  serviceName: nacos-headless
  replicas: 3
  template:
    metadata:
      labels:
        app: nacos
      annotations:
        pod.alpha.kubernetes.io/initialized: "true"
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: "app"
                    operator: In
                    values:
                      - nacos
              topologyKey: "kubernetes.io/hostname"
      serviceAccountName: nfs-client-provisioner
      initContainers:
        - name: peer-finder-plugin-install
          image: nacos/nacos-peer-finder-plugin:1.0
          imagePullPolicy: Always
          volumeMounts:
            - mountPath: "/home/nacos/plugins/peer-finder"
              name: nacos-plugindir
      containers:
        - name: nacos
          imagePullPolicy: Always
          image: nacos/nacos-server:latest
          resources:
            requests:
              memory: "2Gi"
              cpu: "500m"
          ports:
            - containerPort: 8848
              name: client-port
            - containerPort: 7848
              name: rpc
          env:
            - name: NACOS_REPLICAS
              value: "3"
            - name: SERVICE_NAME
              value: "nacos-headless"
            - name: DOMAIN_NAME
              value: "cluster.local"
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.namespace
            - name: MYSQL_SERVICE_DB_NAME
              valueFrom:
                configMapKeyRef:
                  name: nacos-cm
                  key: mysql.db.name
            - name: MYSQL_SERVICE_PORT
              valueFrom:
                configMapKeyRef:
                  name: nacos-cm
                  key: mysql.port
            - name: MYSQL_SERVICE_USER
              valueFrom:
                configMapKeyRef:
                  name: nacos-cm
                  key: mysql.user
            - name: MYSQL_SERVICE_PASSWORD
              valueFrom:
                configMapKeyRef:
                  name: nacos-cm
                  key: mysql.password
            - name: NACOS_SERVER_PORT
              value: "8848"
            - name: NACOS_APPLICATION_PORT
              value: "8848"
            - name: PREFER_HOST_MODE
              value: "hostname"
          volumeMounts:
            - name: nacos-plugindir
              mountPath: /home/nacos/plugins/peer-finder
            - name: nacos-datadir
              mountPath: /home/nacos/data
            - name: nacos-logdir
              mountPath: /home/nacos/logs
  volumeClaimTemplates:
    - metadata:
        name: nacos-plugindir
        annotations:
          volume.beta.kubernetes.io/storage-class: "managed-nfs-storage"
      spec:
        accessModes: [ "ReadWriteMany" ]
        resources:
          requests:
            storage: 5Gi
    - metadata:
        name: nacos-datadir
        annotations:
          volume.beta.kubernetes.io/storage-class: "managed-nfs-storage"
      spec:
        accessModes: [ "ReadWriteMany" ]
        resources:
          requests:
            storage: 5Gi
    - metadata:
        name: nacos-logdir
        annotations:
          volume.beta.kubernetes.io/storage-class: "managed-nfs-storage"
      spec:
        accessModes: [ "ReadWriteMany" ]
        resources:
          requests:
            storage: 5Gi
  selector:
    matchLabels:
      app: nacos
```

上述配置文件中指定的`storageclass`为`managed-nfs-storage`，这个名为`managed-nfs-storage`是不存在的，我创建的`storageclass`的名称是`nfs-storage`。

因此，当使用`kubectl create -f nacos-k8s/deploy/nacos/nacos-pvc-nfs.yaml`创建的时候，pvc将处于`pending`状态。

```shell
[root@k8s-master nacos-k8s]# kubectl get pvc
NAME                      STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS          AGE
nacos-datadir-nacos-0     Pending                                      managed-nfs-storage   149m
nacos-logdir-nacos-0      Pending                                      managed-nfs-storage   149m
nacos-plugindir-nacos-0   Pending                                      managed-nfs-storage   149m
```

这时正常人都会想到`kubectl delete -f nacos-k8s/deploy/nacos/nacos-pvc-nfs.yaml`删除资源，修改storageclass成正确的之后再重新create。但是这样做的结果就是nacos无法启动，也一直pending，通过`kubectl describe pod nacos-o`可以看到有`running "VolumeBinding" filter plugin for pod "nacos-0": pod has unbound immediate PersistentVolumeClaims`的错误提示，查看pod状态如下：

```shell
[root@k8s-master nacos-k8s]# kubectl get pod -o wide
NAME                                     READY   STATUS    RESTARTS   AGE     IP              NODE        NOMINATED NODE   READINESS GATES
mysql-59szx                              1/1     Running   0          40m     10.109.131.62   k8s-node2   <none>           <none>
nacos-0                                  0/1     Pending   0          6m10s   <none>          <none>      <none>           <none>
nfs-client-provisioner-6965c6967-z597p   1/1     Running   0          13m     10.111.218.68   k8s-node3   <none>           <none>
```

问题的关键在于名为`nacos-datadir-nacos-0` `nacos-logdir-nacos-0` `nacos-plugindir-nacos-0 `的pvc并没有被删除，一直pending，所以pod也跟着pending，解决办法即手动删除这些pending的pvc。