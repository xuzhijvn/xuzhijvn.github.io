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

woocommerce_rest_authentication_error

nginx配置错误导致woocommerce REST API不可用

It was a bad configuration of try_files in nginx:

WRONG

```shell
try_files $uri $uri/ /index.php?q=$uri&$args;
```

CORRECT

```shell
try_files $uri $uri/ /index.php$is_args$args;
```

After changing that everything works perfectly ^^

参考：
[woocommerce_rest_authentication_error on localhost](https://github.com/woocommerce/woocommerce/issues/20815)

