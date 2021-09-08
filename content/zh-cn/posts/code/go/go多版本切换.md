---
title: "go多版本切换"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"GO"
]
series : [
"Manual"
]
images : [
"images/center.png"
]
---

[comment]: <> (# go多版本切换)



## 方法一：调整PATH

```sh
export PATH=/usr/local/go/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin 
```

![image-20210626133140778](https://picgo.6and.ltd/img/image-20210626133140778.png)



## 方法二：调整软链接(未验证)

```
//查看当前软链接指向
cd /usr/local/bin 
ls -trl | grep go
//调整软链接
ln -snf /usr/local/Cellar/go/1.16.3/bin go
```

