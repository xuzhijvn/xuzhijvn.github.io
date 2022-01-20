---
title: "本地连接k8s集群"
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

# 本地连接k8s集群

在日常开发的过程中，经常会需要在本地开发的程序需要在k8s中调试的场景，比如，写了一个operator。

如果此时，本地又没有可以直接可达的k8s集群，

比如k8s是在公有云的vpc环境内，外面无法直接访问。想要本地连接远程k8s集群，可以参考 [本地连接远程的内网k8s集群](http://kuring.me/post/local-connect-remote-k8s/)

再比如k8s集群是自己通过多台云服务器自行搭建的，master节点有自己的公网ip。想要本地连接远程k8s集群，可以参考本文。



## 1⃣️ 重新生成config文件

默认下，`~/.kube/config` 生成配置文件的时候只包含了k8s集群ip和这个节点的局域网ip，本地如果想远程操作k8s的话，必定要通过公网ip连接到k8s集群，所以我们需要把节点绑定的公网ip也放到证书里面去，即我们需要重新生成证书。如果不这样做，本地直接访问的话，会报如下提示：

```bash
tony@192 ~ % kubectl get pod 
Unable to connect to the server: x509: certificate is valid for 10.96.0.1, 172.17.0.14, not 106.55.152.92
```

先备份证书：

```bash
[root@k8s-master .kube]# mkdir -p /etc/kubernetes/pki.bak
[root@k8s-master .kube]# mv /etc/kubernetes/pki/apiserver.* /etc/kubernetes/pki.bak
```

重新生成证书：

```bash
[root@k8s-master .kube]# kubeadm init phase certs all --apiserver-advertise-address=0.0.0.0 --apiserver-cert-extra-sans=10.96.0.1,172.17.0.14,xxx.xxx.xxx.xxx(公网ip)
I0627 15:10:39.069106    7777 version.go:252] remote version is much newer: v1.21.2; falling back to: stable-1.18
W0627 15:10:40.982380    7777 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Using existing ca certificate authority
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 172.17.0.14 10.96.0.1 172.17.0.14 106.55.152.92]
[certs] Using existing apiserver-kubelet-client certificate and key on disk
[certs] Using existing front-proxy-ca certificate authority
[certs] Using existing front-proxy-client certificate and key on disk
[certs] Using existing etcd/ca certificate authority
[certs] Using existing etcd/server certificate and key on disk
[certs] Using existing etcd/peer certificate and key on disk
[certs] Using existing etcd/healthcheck-client certificate and key on disk
[certs] Using existing apiserver-etcd-client certificate and key on disk
[certs] Using the existing "sa" key
```

重启apiserver:

```bash
[root@k8s-master .kube]# docker rm -f `docker ps -q -f 'name=k8s_kube-apiserver*'`
cd130929bc7e
[root@k8s-master .kube]# systemctl restart kubelet
```

## 2⃣️ 修改ip

拷贝master节点的配置文件 `~/.kube/config` 到本地 `~/.kube/config` ，修改server的ip地址为master节点的公网ip地址

![image-20210627152937670](https://picgo.6and.ltd/img/image-20210627152937670.png)



## 3⃣️ 本地验证

```bash
tony@192 ~ % kubectl get pod
NAME                                                 READY   STATUS    RESTARTS   AGE
ceres-admin-server-deployment-7f9c4c697d-7pphh       2/2     Running   0          92d
ceres-admin-web-deployment-7c8fb64f58-ft8xv          2/2     Running   2          192d
ceres-app-server-deployment-65b6bb99f9-qlwb2         2/2     Running   0          92d
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
```

### 参考链接🔗

[Invalid x509 certificate for kubernetes master](https://stackoverflow.com/questions/46360361/invalid-x509-certificate-for-kubernetes-master)

