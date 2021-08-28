---
title: "去除请求path前缀"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"nginx"
]
series : [
"Manual"
]
images : [
"images/center.png"
]
---

[comment]: <> (# 去除请求path前缀)

## 1. proxy_pass后面加根路径`/`

```shell
location ^~/user/ {
    proxy_set_header Host $host;
    proxy_set_header  X-Real-IP        $remote_addr;
    proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
    proxy_set_header X-NginX-Proxy true;

    proxy_pass http://user/;
}
```

`^~/user/`表示匹配前缀是`user`的请求，proxy_pass的结尾有`/`， 则会把`/user/*`后面的路径直接拼接到后面，即移除user。

## 2. 使用`rewrite`

```shell
location ^~/user/ {
    proxy_set_header Host $host;
    proxy_set_header  X-Real-IP        $remote_addr;
    proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
    proxy_set_header X-NginX-Proxy true;

    rewrite ^/user/(.*)$ /$1 break;
    proxy_pass http://user;
}
```

注意到proxy_pass结尾没有`/`， `rewrite`重写了url。

### 参考链接：

[Nginx代理proxy pass配置去除前缀](https://www.cnblogs.com/woshimrf/p/nginx-proxy-rewrite-url.html)

[Nginx 转发域名地址报 400 Bad Request](https://priesttomb.github.io/技术/2020/05/05/nginx-400-error-about-host/)
