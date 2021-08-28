---
title: "Ubuntu修改域名解析"
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

nginx 访问 .php文件变成下载(chrome) 或者直接显示源码(edge)

原因：这是因为nginx没有设置好碰到php文件时，要传递到后方的php解释器。

需要在nginx.conf的server{}添加如下内容：

```shell
    location ~ [^/]\.php(/|$) {
      #fastcgi_pass remote_php_ip:9000;
      fastcgi_pass unix:/dev/shm/php-cgi.sock;
      fastcgi_index index.php;
      include fastcgi.conf;
    }
```

 

 

