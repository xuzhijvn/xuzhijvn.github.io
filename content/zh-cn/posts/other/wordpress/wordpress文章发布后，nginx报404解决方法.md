---
title: "wordpress文章发布后，nginx报404解决方法"
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

wordpress文章发布后，nginx报404解决方法

修改nginx.conf文件，在location /节点下添加如下代码：

```nginx
location / {        
    try_files $uri $uri/ /index.php?q=$uri&$args; 
}
```

然后重启nginx即可解决。

