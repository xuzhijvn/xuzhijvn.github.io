---
title: "阿里云下配置MySQL远程连接"
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

[comment]: <> (# 阿里云下配置MySQL远程连接)

#### 1）use mysql

#### 2）将host设置为%表示任何ip都能连接mysql

```shell
update user set host='%' where user='root' and host='localhost';
```



#### 3) 执行完以上语句,接着执行以下语句 ,刷新权限表,使配置生效

```shell
flush privileges;
```



### 参考链接：

[阿里云下配置MySQL远程连接的步骤详解](https://www.jb51.net/article/121173.htm)
