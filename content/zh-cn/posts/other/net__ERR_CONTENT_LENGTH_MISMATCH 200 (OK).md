---
title: "net::ERR_CONTENT_LENGTH_MISMATCH 200 (OK)"
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

net::ERR_CONTENT_LENGTH_MISMATCH 200 (OK)

加载静态资源时报错：`net::ERR_CONTENT_LENGTH_MISMATCH 200 (OK)`

![img](https://picgo.6and.ltd/img/img_5fa02506e21da.png)

解决办法：调整缓冲区大小 `proxy_buffer_size 64k; proxy_buffers 4 32k; proxy_busy_buffers_size 64k;`

```shell
server {

        listen 80;
        server_name tkaid.com www.tkaid.com;
        location / {
            proxy_buffer_size 64k;
            proxy_buffers   4 32k;
            proxy_busy_buffers_size 64k;
            proxy_pass http://k8s_tkb-web-svc;
        }
}
```

如果仍然报这个错，可以再将值设置得大一点。

