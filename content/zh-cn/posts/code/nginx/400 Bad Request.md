---
title: "400 Bad Request"
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

]
---

[comment]: <> (# 400 Bad Request)

需要设置 `proxy_set_header   Host $host:$server_port;`

```shell
location ^~/gateway/ {
    proxy_set_header   Host $host:$server_port;
    proxy_set_header   X-Real-IP $remote_addr;
    proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header   X-NginX-Proxy true;
    proxy_pass http://k8s_yshop-gateway-svc/;
}
```

 

