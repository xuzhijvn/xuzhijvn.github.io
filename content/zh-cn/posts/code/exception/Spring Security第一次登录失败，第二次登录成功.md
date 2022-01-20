---
title: "Spring Security第一次登录失败，第二次登录成功"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"spring"
]
series : [
"Manual"
]
images : [

]
---

[comment]: <> (# Spring Security第一次登录失败，第二次登录成功)


当我第一次登录时我会得到`{" timestamp"：1481719982036，" status"：999，" error"：" None"，" message"："无可用消息"}`，但第二次还可以。

解决办法：填写如下配置到application.properties

```properties
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration
```

参考链接：[java:Spring Security第一次登录失败，第二次登录成功](https://codebug.vip/questions-1787119.htm)

