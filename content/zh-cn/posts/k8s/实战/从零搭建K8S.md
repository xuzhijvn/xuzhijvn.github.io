---
title: "从零搭建K8S"
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


从零搭建K8S

## 机器准备

| 名称   | 数量 | IP          | 备注                                                         |
| ------ | ---- | ----------- | ------------------------------------------------------------ |
| master | 1    | 172.17.0.14 | 操作系统: Linux(centos7, 其它操作系统也可, 安装过程类似, 可参考官方文档) 机器配置: 4C8G |
| node1  | 1    | 172.18.0.7  | 同上                                                         |
| node2  | 1    | 172.19.0.5  | 同上                                                         |

由于本人很穷，这几台机器是分别属于不同的腾讯云账号，不同的账号之间不能内网通信，不过可以通过建立“对等连接”实现通信，比直接用公网通信靠谱。

## 1. 修改hostname

```shell
[root@k8s-master ~]$ vim /etc/hostname # 修改hostname
[root@k8s-master ~]$ vim /etc/hosts	# 将本机IP指向hostname
[root@k8s-master ~]$ reboot -h 		# 重启(可以做完全部前期准备后再重启)
```

修改后：

```shell
[root@k8s-master ~]# cat /etc/hosts

::1 VM_0_5_centos VM_0_5_centos
::1 localhost.localdomain localhost
::1 localhost6.localdomain6 localhost6

127.0.0.1 localhost localhost.localdomain k8s-master
172.17.0.14 k8s-master
172.18.0.7 k8s-node1
172.19.0.5 k8s-node2
```

## 2. 配置防火墙

笔者图方便, 直接关闭了防火墙. 若安全要求较高, 可以参考官方文档放行必要端口.

```shell
[root@k8s-master ~]$ systemctl stop firewalld	# 关闭服务
[root@k8s-master ~]$ systemctl disable firewalld	# 禁用服务
```

## 3. 禁用SELinux

腾讯云centos7.6默认是禁止的，如果你的不是，请修改`/etc/selinux/config`, 设置SELINUX=disabled. 重启机器.

```shell
[root@k8s-master ~]$ sestatus	# 查看SELinux状态
SELinux status: disabled
```

## 4. 禁用交换分区

腾讯云centos7.6默认是禁止的，如果你的不是，请编辑`/etc/fstab`, 将swap注释掉. 重启机器.

```shell
[root@k8s-master ~]$ vim /etc/fstab 
#/dev/mapper/cl-swap     swap                    swap    defaults        0 0
```

## 5. 安装Docker

方法一： 官方安装脚本自动安装

```shell
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

方法二： 手动安装

```shell
//第一步

yum install -y yum-utils \ device-mapper-persistent-data \ lvm2

//第二步

yum-config-manager \ --add-repo \ http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

//第三步

yum install docker-ce docker-ce-cli containerd.io

//第四步

systemctl start docker
```

 

配置docker：

```shell
[root@k8s-master ~]# cat /etc/docker/daemon.json 
{
   "registry-mirrors": [ "https://mirror.ccs.tencentyun.com"],
   "exec-opts": ["native.cgroupdriver=systemd"],
   "bip": "172.200.0.1/16"
}
```

安装配置完毕后执行:

```shell
[root@k8s-master ~]$ systemctl enable docker
[root@k8s-master ~]$ systemctl start docker
```

## 6. 安装Kubernetes

由于国内网络原因, 官方文档中的地址不可用, 本文替换为阿里云镜像地址, 执行以下代码即可:

```shell
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

安装kubeadm，kubelet，kubectl：

```shell
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
```

修改网络配置：

```shell
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

[su_highlight]注意: 至此, 以上的全部操作, 在Worker机器上也需要执行. 注意hostname等不要相同.[/su_highlight]

## 7. 初始化Master

```shell
[root@k8s-master ~]$ kubeadm config print init-defaults > kubeadm-init.yaml
```

该文件有两处需要修改:

将`advertiseAddress`: 1.2.3.4修改为本机地址
将`imageRepository`: k8s.gcr.io修改为imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 172.17.0.14
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8s-master
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.15.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

下载镜像：

```shell
[root@k8s-master ~]$ kubeadm config images pull --config kubeadm-init.yaml
```

执行初始化：

```shell
[root@k8s-master ~]$ kubeadm init --config kubeadm-init.yaml
```

等待执行完毕后, 会输出如下内容：

```shell
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.17.0.14:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:e245251e3de01986694f77319827481ed8669be6ba2ccc23a29596072b275346
```

最后两行需要保存下来, kubeadm join ...是worker节点加入所需要执行的命令.

接下来配置环境, 让当前用户可以执行kubectl命令:

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

测试一下: 此处的NotReady是因为网络还没配置.

```shell
[root@k8s-master kubernetes]$ kubectl get node
NAME         STATUS     ROLES    AGE     VERSION
k8s-master   NotReady   master   3m25s   v1.15.3
```

## 8. 配置网络

下载描述文件：

```shell
[root@k8s-master ~]$ wget https://docs.projectcalico.org/v3.15/manifests/calico.yaml
[root@k8s-master ~]$ cat kubeadm-init.yaml | grep serviceSubnet:
serviceSubnet: 10.96.0.0/12
```

打开calico.yaml, 将`192.168.0.0/16`修改为`10.96.0.0/12`

![img](https://picgo.6and.ltd/img/img_5ef33afe955d8-20210621122448233.png)

[su_highlight]需要注意的是, `calico.yaml`中的IP和`kubeadm-init.yaml`需要保持一致, 要么初始化前修改`kubeadm-init.yaml`, 要么初始化后修改`calico.yaml`.[/su_highlight]

执行`kubectl apply -f calico.yaml`初始化网络.

此时查看node信息, master的状态已经是Ready了.

```shell
[root@k8s-master ~]$ kubectl get node
NAME         STATUS   ROLES    AGE   VERSION
k8s-master   Ready    master   15m   v1.15.3
```

## 9. 安装Dashboard

### 91. 部署Dashboard

```shell
[root@k8s-master ~]$ wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.3/aio/deploy/recommended.yaml
[root@k8s-master ~]$ kubectl apply -f recommended.yaml
```

部署完毕后, 执行`kubectl get pods --all-namespaces`查看pods状态

```shell
[root@k8s-master kubernetes]$ kubectl get pods --all-namespaces | grep dashboard
NAMESPACE              NAME                                        READY   STATUS   
kubernetes-dashboard   dashboard-metrics-scraper-fb986f88d-m9d8z   1/1     Running
kubernetes-dashboard   kubernetes-dashboard-6bb65fcc49-7s85s       1/1     Running
```

### 9.2 创建用户

创建一个用于登录Dashboard的用户. 创建文件`dashboard-adminuser.yaml`内容如下:

```shell
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```

执行命令`kubectl apply -f dashboard-adminuser.yaml`.

### 9.3 生成证书

```shell
[root@k8s-master ~]$ grep 'client-certificate-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.crt
[root@k8s-master ~]$ grep 'client-key-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.key
[root@k8s-master ~]$ openssl pkcs12 -export -clcerts -inkey kubecfg.key -in kubecfg.crt -out kubecfg.p12 -name "kubernetes-client"
```

第三条命令生成证书时会提示输入密码, 可以直接两次回车跳过.

`kubecfg.p12`即需要导入客户端机器的证书. 将证书拷贝到客户端机器上, 导入即可. chrome浏览器按下图所示导入：

![img](https://picgo.6and.ltd/img/img_5ef33c7e9db84-20210621122443116.png)

此时我们可以登录面板了, 访问地址: `https://{k8s-master-ip}:6443/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login`

![img](https://picgo.6and.ltd/img/img_5ef33d17328d4.png)



 



![img](https://picgo.6and.ltd/img/img_5ef33d20519c2.png)



### 9.4 登录Dashboard

执行`kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')`, 获取Token.

```shell
[root@k8s-master ~]# kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
Name:         admin-user-token-6mpx4
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: cabcc858-826a-4236-8514-51f473bf7752

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6Iks2dmRwalB5SWNKbWJTVUUxanVlVlAwbTk1OHR6QzhfN0FOZUw0V3huM0UifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLTZtcHg0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJjYWJjYzg1OC04MjZhLTQyMzYtODUxNC01MWY0NzNiZjc3NTIiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.jHAzmmOBRn5tfZBeukORLm--9_q879fTHPDzjjyCm42MbHP0TFrgbo2A8ZmN0Od4qe9rTHiD9pJl3BtUQ06pIMsly7LvENnuLAxGuK3oDqc5FCdqb8L-f9-9HmBo7nEAMy67i6Cv9TjfD9790ejfOg6ZI0PC8MDXInxSgY97hlwBlyJh_M5zz8SMnOOnMYyr8UFmHRZJjTO5pdIs7cdVBxLz27wzw4h0svJyOsi9MBqHskN6Hq7KOsEYP5wyDHXmU_iHqQJH64R9DA6dpHx6v3qV1dyhtXJE9tZECsvoVtvNlKQ-VirtX4sJX29a5wAXDqkcysDiClIXZGZKtg8tFw
```

复制该Token到登录页, 点击登录即可, 效果如下:

![img](https://picgo.6and.ltd/img/img_5ef33d9fd5224-20210621122422540.png)

## 10. 添加Node节点

执行如下命令将Worker加入集群:

```shell
kubeadm join 172.17.0.14:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:e245251e3de01986694f77319827481ed8669be6ba2ccc23a29596072b275346
```

注意: 此处的秘钥是初始化Master后生成的, 参考前文.

如果token过期，使用如下命令重新生成：

```shell
kubeadm token create --print-join-command
```

参考连接：[kubeadm 生成的token过期后，集群增加节点](https://www.cnblogs.com/hongdada/p/9854696.html)

如果join时报错：`[ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1` 执行如下命令：

```shell
echo "1">/proc/sys/net/ipv4/ip_forward
```

参考链接：[kubernetes 加入子节点](https://learnku.com/articles/35188)

添加完毕后, 在Master上查看节点状态:

```shell
[root@k8s-master ~]# kubectl get node
NAME         STATUS   ROLES    AGE    VERSION
k8s-master   Ready    master   125m   v1.18.4
k8s-node1    Ready    <none>   97m    v1.18.4
k8s-node2    Ready    <none>   95m    v1.18.4
```

在面板上也可查看:

![img](https://picgo.6and.ltd/img/img_5ef33e6d5addd-20210621122430349.png)

配置node节点，以便node节点能够执行类似`kubectl get node`的时候不至于报`The connection to the server localhost:8080 was refused - did you specify the right host or port?`

在node节点上执行：

```shell
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/kubelet.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config //如果你本身是root用户，可以不执行
```

 

## 参考链接

https://juejin.im/post/5d7fb46d5188253264365dcf