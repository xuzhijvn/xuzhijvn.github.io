---
title: "Jenkins使用使用注意事项"
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

]
---

Jenkins使用使用注意事项

## 1. 无法通过execute shell启动进程

这是因为Jenkins默认会在Build结束后Kill掉所有的衍生进程。

在执行shell前需要设置BUILD_ID=dontKillMe

```shell
BUILD_ID=dontKillMe
cd /data/wwwroot/yxshop
chmod +x ./*.sh
nohup ./restart.sh >/data/wwwroot/yxshop/nohup.out 2>&1 &
```

## 2. github向jenkins deliver失败

一开始填写的Payload URL形如，

Payload URL = https://xxx.com/github-webhook

无论怎么修改再redeliver都失败，后来修改为形如，

Payload URL = https://xxx.com/github-webhook/

仅仅是在后面加了“/”。

