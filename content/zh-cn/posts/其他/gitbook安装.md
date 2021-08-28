---
title: "gitbook安装"
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
"images/center.png"
]
---

gitbook安装

1. 安装

```shell
sudo npm install gitbook -g
sudo npm install -g gitbook-cli
```

2. 验证

```shell
gitbook -V
```

3. 初始化项目

```shell
mkdir direName //创建自己的文件夹目录
cd direName    //进入到自己的gitbook文件夹目录
gitbook init   //初始化gitbook项目
```

4. 启动

```shell
gitbook serve
```

或者`gitbook serve &`后台运行，默认在4000端口启动。

