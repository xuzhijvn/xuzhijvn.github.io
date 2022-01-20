---
title: "部署vue项目"
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

[comment]: <> (# 部署vue项目)

时常我们想通过path来区分项目，例如通过 `http://xxxx/admin` 访问我们的后台，如果vue是的`mode`是`history`，请注意如下配置：

1. 修改vue-config.js文件配置

```vue
module.exports = {publicPath: ""};
```

2. 修改路由route/index

```vue
const router = new Router({  
    base: '/admin/',  //路由模式为history模式时，base必须要加上;路由模式为hash模式时，base可加可不加  
    mode: 'history',
    routes: []
}
```

此时，请求是形如 `http://xxxx/admin/xxx.css`，从而避免出现形如`http://xxxx/xxx.css`找不到资源的情况。

