---
title: "Ubuntu修改ip"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"其他"
]
tags : [
"Other"
]
series : [
"Manual"
]
images : [

]
---

Ubuntu修改ip

### 方式一

这种方式可以修改ip地址，不能访问互联网；虚拟机100必须以这种方式配置（101、102、103是复制100而来）



1、sudo vi /etc/netplan/50-cloud-init.yaml

```yaml
network:
    ethernets:
        enp0s3:
            addresses: [10.0.2.15/24]
            dhcp4: true
	enp0s8:
            addresses: [192.168.56.100/24]
            dhcp4: false
    version: 2
```

2、重启虚拟机

### 方式二

这种方式可以修改ip地址，能访问互联网；虚拟机101、102、103以这种方式配置



1、注释50-cloud-init.yaml里面的修改



2、sudo vim /etc/network/interfaces

```shell
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface(NAT)
auto enp0s3
iface enp0s3 inet dhcp

# 增加的Host-only静态IP设置 (enp0s8 是根据拓扑关系映射的网卡名称（旧规则是eth0,eth1）)
# 可以通过 ```ls /sys/class/net```查看，是否为enp0s8
auto enp0s8
iface enp0s8 inet static
address 192.168.56.101
netmask 255.255.255.0
```
