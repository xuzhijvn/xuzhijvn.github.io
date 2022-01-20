---
title: "nfs安装"
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

nfs安装

## 1. 环境说明

| 角色       | ip          | os         |
| ---------- | ----------- | ---------- |
| NFS 服务端 | 172.31.0.12 | CentOS 7.6 |
| NFS 客户端 | 172.18.0.7  | CentOS 7.6 |

## 2. NFS 服务端

### 2.1 安装 nfs-utils

```shell
# rpcbind 作为依赖会自动安装。
> yum install nfs-utils
```

### 2.2 配置并启动服务

允许rpcbind.service、nfs.service开机自启：

```shell
# 启动相关服务
> systemctl start rpcbind
> systemctl start nfs
```

防火墙允许服务通过：

```shell
# 防火墙允许服务通过
> firewall-cmd --zone=public --permanent --add-service={rpc-bind,mountd,nfs}
success

> firewall-cmd --reload
success
```

或者直接关闭防火墙：`systemctl stop firewalld`

### 2.3 配置共享目录

例如需要共享的目录为`/mnt/nfs/`：

```shell
# 创建 /mnt/kvm 并修改权限
> cd /mnt
/mnt > mkdir nfs
/mnt > chmod 755 nfs

# 验证目录权限
/mnt > ls -l
total 0
drwxr-xr-x 2 root root 59 Oct 17 17:49 nfs
```

之后修改`/etc/exports`，将`/mnt/nfs/`添加进去：

```shell
> cat /etc/exports

# 1. 只允许 abelsu7-ubuntu 访问
/mnt/nfs/ abelsu7-ubuntu(rw,sync,no_root_squash,no_all_squash)

# 2. 根据 IP 地址范围限制访问
/mnt/nfs/ 172.0.0.0/8(rw,sync,no_root_squash,no_all_squash)

# 3. 使用 * 表示访问不加限制
/mnt/nfs/ *(rw,sync,no_root_squash,no_all_squash)
```

关于`/etc/exports`中的**参数含义**：

- `/mnt/kvm/`：需要**共享的目录**
- `172.0.0.0/8`：**客户端 IP 范围**，`*`表示无限制
- `rw`：**权限设置**，可读可写
- `sync`：表示文件同时写入硬盘和内存
- `no_root_squash`：当登录 NFS 主机使用共享目录的使用者是 root 时，其权限将被转换成为匿名使用者，通常它的 UID 与 GID，都会变成 nobody 身份
- `no_all_squash`：可以使用**普通用户授权（？？）**

保存之后，重启`nfs`服务：

```shell
> systemctl restart nfs
```

### 2.4 查看共享目录列表

```shell
[root@VM-0-12-centos mnt]# showmount -e localhost
Export list for localhost:
/mnt/nfs 172.0.0.0/8
```

## 3. NFS 客户端

### 3.1 安装 nfs-utils

```shell
# CentOS/Fedora, etc.
> yum install nfs-utils

# Ubuntu/Debian, etc.
> apt install nfs-common
```

必须要安装nfs-utils，安装过程会下载关键命令到本机，例如`showmount`等等。

### ~~3.2 配置并启动服务（非必须步骤）~~

> 开启`rpcbind`服务不是必须的，经过实测只要安装了nfs-utils到本机，就可以实现挂载。
>

设置`rpcbind`服务开机启动：

```shell
> systemctl enable rpcbind
```

启动`rpcbind`：

```shell
> systemctl start rpcbind
```

客户端不需要打开防火墙，也不需要开启 NFS 服务

### 3.3 挂载共享目录

先查看服务端的共享目录：

```shell
[root@k8s-node1 mnt]# showmount -e 172.31.0.12
Export list for 172.31.0.12:
/mnt/nfs 172.0.0.0/8
```

在客户端创建并挂载对应目录：

```shell
> mkdir -p /mnt/nfs
> mount -t nfs 172.31.0.12:/mnt/nfs /mnt/nfs
```

最后检查一下是否挂载成功：

```shell
[root@k8s-node1 mnt]# df -hT /mnt/nfs
Filesystem           Type  Size  Used Avail Use% Mounted on
172.31.0.12:/mnt/nfs nfs4  493G   72M  467G   1% /mnt/nfs
[root@k8s-node1 mnt]#  mount | grep /mnt/nfs
172.31.0.12:/mnt/nfs on /mnt/nfs type nfs4 (rw,relatime,vers=4.1,rsize=524288,wsize=524288,namlen=255,hard,proto=tcp,port=0,timeo=600,retrans=2,sec=sys,clientaddr=172.18.0.7,local_lock=none,addr=172.31.0.12)
```

### 3.4 配置自动挂载

在客户端编辑/etc/fstab，加入如下配置：

```shell
172.31.0.12:/mnt/nfs                 /mnt/nfs                nfs       defaults              0 0
```

最后重新加载`systemctl`，即可实现重启后自动挂载

```shell
> systemctl daemon-reload
```

## 4. NFS 读写速度测试

写速度：

```shell
[root@k8s-node1 mnt]# time dd if=/dev/zero of=/mnt/nfs/jenkins.war bs=8k count=1024
1024+0 records in
1024+0 records out
8388608 bytes (8.4 MB) copied, 0.122319 s, 68.6 MB/s

real	0m0.138s
user	0m0.000s
sys	0m0.008s
```

读速度：

```shell
[root@k8s-node1 mnt]# time dd if=/mnt/nfs/jenkins.war of=/dev/null bs=8k count=1024
1024+0 records in
1024+0 records out
8388608 bytes (8.4 MB) copied, 0.0020144 s, 4.2 GB/s

real	0m0.004s
user	0m0.000s
sys	0m0.002s
```

### 参考连接

[CentOS 7 安装配置 NFS](https://abelsu7.top/2019/10/17/centos7-install-nfs/#1-环境说明)
