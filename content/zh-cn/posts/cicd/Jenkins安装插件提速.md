---
title: "Jenkins安装插件提速"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"持续集成部署"
]
tags : [
"Jenkins"
]
images : [
"images/center.png"
]
---

Jenkins安装插件提速

国内安装Jenkins插件缓慢，这是我见到最完美的解决方案。
通过【插件-->高级-->更新网站】替换成清华的数据源（https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json）并不好使

![img](https://picgo.6and.ltd/img/img_5e9b36bba6599.png)
需要按照如下步骤操作：

## 进入到工作目录

```shell
$ cd {你的Jenkins工作目录}/updates  #进入更新配置位置
```

## 修改default.json

### 方法1

```shell
$ vim default.json
```

替换所有插件下载的url

```shell
:1,$s/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g
```

替换连接测试url

```shell
:1,$s/http:\/\/www.google.com/https:\/\/www.baidu.com/g
```

**进入vim先输入：然后再粘贴上边的：后边的命令，注意不要写两个冒号！**

修改完成保存退出:wq

### 方法2

使用sed

```shell
$ sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json
```

**这是直接修改的配置文件，如果前边Jenkins用sudo启动的话，那么这里的两个sed前均需要加上sudo**

重启Jenkins，安装插件试试，简直超速！！

https://www.cnblogs.com/hellxz/p/jenkins_install_plugins_faster.html