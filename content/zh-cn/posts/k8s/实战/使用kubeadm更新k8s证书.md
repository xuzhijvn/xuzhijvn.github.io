---
title: "使用kubeadm更新k8s证书"
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


# 使用kubeadm更新k8s证书

今天操作k8s的时候，突然说证书无效：

```shell
Unable to connect to the server: x509: certificate has expired or is not yet valid
```

通过 `kubeadm alpha certs check-expiration` 查看，确实是过期了：

```shell
[root@k8s-master ~]# kubeadm alpha certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[check-expiration] Error reading configuration from the Cluster. Falling back to default configuration

W0627 11:21:35.745166    8754 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Jun 24, 2021 09:45 UTC   <invalid>                               no      
apiserver                  Jun 24, 2021 09:45 UTC   <invalid>       ca                      no      
apiserver-etcd-client      Jun 24, 2021 09:45 UTC   <invalid>       etcd-ca                 no      
apiserver-kubelet-client   Jun 24, 2021 09:45 UTC   <invalid>       ca                      no      
controller-manager.conf    Jun 24, 2021 09:45 UTC   <invalid>                               no      
etcd-healthcheck-client    Jun 24, 2021 09:45 UTC   <invalid>       etcd-ca                 no      
etcd-peer                  Jun 24, 2021 09:45 UTC   <invalid>       etcd-ca                 no      
etcd-server                Jun 24, 2021 09:45 UTC   <invalid>       etcd-ca                 no      
front-proxy-client         Jun 24, 2021 09:45 UTC   <invalid>       front-proxy-ca          no      
scheduler.conf             Jun 24, 2021 09:45 UTC   <invalid>                               no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Jun 22, 2030 09:45 UTC   8y              no      
etcd-ca                 Jun 22, 2030 09:45 UTC   8y              no      
front-proxy-ca          Jun 22, 2030 09:45 UTC   8y              no   
```

那么接下来就是更新证书📄了：

> 下面的操作都是在 master 节点上进行

1. ###### 备份

```shell
//首先备份原有证书：
[root@k8s-master ~]# mkdir /etc/kubernetes.bak
[root@k8s-master ~]# cp -r /etc/kubernetes/pki/ /etc/kubernetes.bak
[root@k8s-master ~]# cp /etc/kubernetes/*.conf /etc/kubernetes.bak
//然后备份 etcd 数据目录：
[root@k8s-master ~]# cp -r /var/lib/etcd /var/lib/etcd.bak
```

2. ###### 通过 `kubeadm alpha certs renew all` 更新证书

```shell
[root@k8s-master ~]# kubeadm alpha certs renew all
[renew] Reading configuration from the cluster...
[renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[renew] Error reading configuration from the Cluster. Falling back to default configuration

W0627 13:06:09.974228   14219 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
certificate for serving the Kubernetes API renewed
certificate the apiserver uses to access etcd renewed
certificate for the API server to connect to kubelet renewed
certificate embedded in the kubeconfig file for the controller manager to use renewed
certificate for liveness probes to healthcheck etcd renewed
certificate for etcd nodes to communicate with each other renewed
certificate for serving etcd renewed
certificate for the front proxy client renewed
certificate embedded in the kubeconfig file for the scheduler manager to use renewed
```

3. ###### 再次查看证书时间

```shell
[root@k8s-master ~]# kubeadm alpha certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Jun 27, 2022 05:06 UTC   364d                                    no      
apiserver                  Jun 27, 2022 05:06 UTC   364d            ca                      no      
apiserver-etcd-client      Jun 27, 2022 05:06 UTC   364d            etcd-ca                 no      
apiserver-kubelet-client   Jun 27, 2022 05:06 UTC   364d            ca                      no      
controller-manager.conf    Jun 27, 2022 05:06 UTC   364d                                    no      
etcd-healthcheck-client    Jun 27, 2022 05:06 UTC   364d            etcd-ca                 no      
etcd-peer                  Jun 27, 2022 05:06 UTC   364d            etcd-ca                 no      
etcd-server                Jun 27, 2022 05:06 UTC   364d            etcd-ca                 no      
front-proxy-client         Jun 27, 2022 05:06 UTC   364d            front-proxy-ca          no      
scheduler.conf             Jun 27, 2022 05:06 UTC   364d                                    no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Jun 22, 2030 09:45 UTC   8y              no      
etcd-ca                 Jun 22, 2030 09:45 UTC   8y              no      
```

已经更新成功了。

4. ###### 查看`kubectl`是否可用

```shell
[root@k8s-master ~]# kubectl get pods
error: You must be logged in to the server (Unauthorized)
```

还不可用，需要更新更新下 kubeconfig 文件。

5. ###### 通过 `kubeadm init phase kubeconfig all` 更新 kubeconfig 文件

```shell
[root@k8s-master ~]# kubeadm init phase kubeconfig all
I0627 13:07:48.893885   15844 version.go:252] remote version is much newer: v1.21.2; falling back to: stable-1.18
W0627 13:07:50.865825   15844 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/admin.conf"
[kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/kubelet.conf"
[kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/controller-manager.conf"
[kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/scheduler.conf"
```

6. ###### 将新生成的 admin 配置文件覆盖掉原本的 admin 文件:

```shell
[root@k8s-master ~]# mv $HOME/.kube/config $HOME/.kube/config.old
[root@k8s-master ~]# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@k8s-master ~]# chown $(id -u):$(id -g) $HOME/.kube/config
```

7. ###### 再次验证证书时间

查看 apiserver 的证书的有效期来验证是否更新成功

```shell
[root@k8s-master ~]# echo | openssl s_client -showcerts -connect 127.0.0.1:6443 -servername api 2>/dev/null | openssl x509 -noout -enddate
notAfter=Jun 27 05:06:10 2022 GMT
```

查看命令是否可用

```shell
[root@k8s-master ~]# kubectl get pod
NAME                                                 READY   STATUS    RESTARTS   AGE
ceres-admin-server-deployment-7f9c4c697d-7pphh       2/2     Running   0          91d
ceres-admin-web-deployment-7c8fb64f58-ft8xv          2/2     Running   2          192d
ceres-app-server-deployment-65b6bb99f9-qlwb2         2/2     Running   0          91d
ceres-jobs-server-deployment-bdcd669bb-284k5         2/2     Running   2          217d
ceres-merchant-web-deployment-6c6b668b8d-5vtq5       2/2     Running   2          192d
ceres-settled-merchant-deployment-5f74c8f46d-f982v   2/2     Running   2          213d
details-v1-6c9f8bcbcb-pzkfm                          2/2     Running   6          296d
nfs-client-provisioner-6965c6967-6jv6c               2/2     Running   8          217d
productpage-v1-7f9d9c48c8-kgtlw                      2/2     Running   4          296d
ratings-v1-65cff55fb8-vmj4z                          2/2     Running   6          296d
reviews-v1-d5b6b667f-9b7kw                           2/2     Running   4          296d
reviews-v2-784495d9bc-xvqpw                          2/2     Running   6          296d
reviews-v3-57fcb844b7-xsj47                          2/2     Running   4          296d
[root@k8s-master ~]# kubens 
default
ingress-nginx
Istio-system
kube-node-lease
kube-public
kube-system
kube-wordpress
kubernetes-dashboard
tkb
```

##### 参考链接🔗：

[更新一个10年有效期的 Kubernetes 证书](https://www.qikqiak.com/post/update-k8s-10y-expire-certs/)

[使用 kubeadm 进行证书管理](https://kubernetes.io/zh/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/)

