---
title: "nginx反向代理kibana报：Kibana did not load properly.Check the server output for more information."
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

nginx反向代理kibana报：Kibana did not load properly.Check the server output for more information.

如题所述，直接访问5601端口不会报错，一旦用ngnix反向代理就报错。

原因：应该是kibana的启动用户没有访问nginx/proxy_temp文件夹的权限，导致部分静态资源无法加载。

> 奇怪的是：1. 我的proxy_temp文件下一个文件都没有 2. 我的nginx error.log日志也没有报类似`Permission denied`的错误。
> 这里暂且不管。。。

解决办法:

```
chmod -R 777 proxy_temp
```

https://www.cnblogs.com/operationhome/p/9901580.html
