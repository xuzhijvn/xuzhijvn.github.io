---
title: "nginx授权登陆报403问题"
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

[comment]: <> (# nginx授权登陆报403问题)

在给nginx加上授权模块后，再访问应用报403访问禁止的错误。

一开始是这样：

### 1. 生成密码文件

```shell
printf "yourusername:$(openssl passwd -apr1)" > /etc/nginx/passwords
```

### 2. nginx配置

```shell
server {
    # ...
    auth_basic "Protected";
    auth_basic_user_file passwords;
    # ...
}
```

[v_error]auth_basic_user_file 后面跟的是相对路径，这样配置很容易导致nginx找不到文件，因此改成绝对路径就万事大吉了：[/v_error]

```shell
server{	
	listen 443 ssl;
	server_name netdata.6and.ltd;
        #listen [::]:81 default_server ipv6only=on;
	#ssl on;
	ssl_certificate httpssl/1_netdata.6and.ltd_bundle.crt;
	ssl_certificate_key httpssl/2_netdata.6and.ltd.key;
	ssl_session_timeout 5m;
	ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
   	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	ssl_prefer_server_ciphers on;
	#index index.html index.htm index.php;
	#root /home/wwwroot/;

	#error_page   404   /404.html;
	#include enable-php.conf;

	auth_basic "Protected";
	auth_basic_user_file /usr/local/nginx/passwords;
   
   	location / {
		proxy_pass http://127.0.0.1:19999;
	}
	access_log  /data/wwwlogs/netdata.log;
}
server {
	listen 80;
	server_name netdata.6and.ltd;
	#把http的域名请求转成https
	return 301 https://$host$request_uri;
}
```

