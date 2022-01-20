---
title: "WordPress 安装插件 cURL error 77解决"
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

WordPress 安装插件 cURL error 77解决

第一步：执行命令

```shell
yum install ca-certificates
```



第二步：重启php-fpm



```shell
/etc/init.d/php-fpm restart
```

