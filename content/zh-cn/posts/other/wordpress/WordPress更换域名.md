---
title: "WordPress更换域名"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"其他"
]
tags : [
"WordPress"
]
series : [
"Manual"
]
images : [

]
---


WordPress更换域名

在更换的域名过程中遇到很多坑，主要还是我的架构比较特殊的原因，导致跟以往配置不太一样，架构如下：

[![img](https://picgo.6and.ltd/img/2020061014034891.png)](http://106.55.152.92:30989/wp-content/uploads/2020/06/2020061014034891.png)

## 1. 无法通过nginx转发请求到容器端口

原因：nginx配置不正确

解决：补充缺失的如下配置

```shell
add_header X-Frame-Options SAMEORIGIN;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_hide_header X-Frame-Options;
```

最终类似：

```shell
server {
    listen 443 ssl;
    listen [::]:443 ssl;
 
    include snippets/ssl-params.conf;
 
    server_name wptest.your-awesome-domain.com;   # domain當然要用自己的，subdomain請隨自己喜好
 
    location / {
        add_header X-Frame-Options SAMEORIGIN;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_hide_header X-Frame-Options;
        proxy_pass http://localhost:8000; # 注意這邊跟上面docker-compose設定的port相同
    }
}
```

## 2. 提示“重定向次数过多”

修改wordpress根目录下的wp-config.php：

```shell
$_SERVER['HTTPS'] = 'on';
define('FORCE_SSL_LOGIN', true);
define('FORCE_SSL_ADMIN', true);
```

 

### 参考链接：

https://softman.blog/2019/07/26/nginx-reverse-proxy-to-dockerized-wordpress-the-basic/
