---
title: "SpringBoot使用QQ邮箱发送邮件配置"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"SpringBoot"
]
series : [
"Manual"
]
images : [

]
---

[comment]: <> (# SpringBoot使用QQ邮箱发送邮件配置)

使用SpringBoot Admin 配置QQ邮箱去发送邮件时报错：com.sun.mail.smtp.SMTPSenderFailedException: 501 mail from address must be same as authorization user，我的配置如下：

```properties
spring.mail.host: smtp.qq.com
spring.mail.username: 发送账号
spring.mail.password: qq授权码
spring.boot.admin.notify.mail.to: 接收账号
```

后来在网上查到是少了spring.boot.admin.notify.mail.from的配置，貌似只有QQ邮箱才需要额外加上这个设置（本人没有测试过用其他邮箱发送邮件）。所以最终配置如下：

```properties
spring.mail.host: smtp.qq.com
spring.mail.username: 发送账号
spring.mail.password: qq授权码
spring.boot.admin.notify.mail.to: 接收账号
spring.boot.admin.notify.mail.from: 发送账号
```

#### 参考：

[Springboot admin 发送邮件失败：com.sun.mail.smtp.SMTPSenderFailedException: 553 Mail from must equal authorized user](http://itren.xiaolee.net/p/1384835.html)

 

