---
title: "Linux配置git账号密码"
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

Linux配置git账号密码

## 1. 在~/下， touch创建文件 .git-credentials, 用vim编辑此文件

```shell
touch .git-credentials
vim .git-credentials
在里面按“i”然后输入： https://{username}:{password}@github.com 

比如 https://account:password@github.com
```

## 2. 在终端下执行

```shell
git config --global credential.helper store
```

## 3. 可以看到~/.gitconfig文件，会多了一项

```shell
[credential]
helper = store
```

 

