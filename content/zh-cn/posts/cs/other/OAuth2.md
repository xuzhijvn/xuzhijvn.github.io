---
title: "OAuth2"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"计算机科学"
]
tags : [
"CS"
]
series : [
"Manual"
]
images : [
"images/center.png"
]
---

[comment]: <> "# OAuth2"

OAuth（开放授权）是一个开放标准，允许用户授权第三方移动应用访问他们存储在另外的服务提供者上的信息，而不需要将用户名和密码提供给第三方移动应用或分享他们数据的所有内容，OAuth2.0是OAuth协议的延续版本，但不向后兼容OAuth 1.0即完全废止了OAuth1.0。

客户端必须得到用户的授权（authorization grant），才能获得令牌（access token）。OAuth 2.0定义了四种授权方式。

- `授权码模式（authorization code）`
- `简化模式（implicit）`
- `密码模式（resource owner password credentials）`
- `客户端模式（client credentials）`

## 1. 名词定义

在详细讲解OAuth 2.0之前，需要了解几个专用名词。

（1）` Third-party application`：第三方应用程序，本文中又称"客户端"（client），即上一节例子中的"云冲印"。

（2）`HTTP service`：HTTP服务提供商，本文中简称"服务提供商"，即上一节例子中的Google。

（3）`Resource Owner`：资源所有者，本文中又称"用户"（user）。

（4）`User Agent`：用户代理，本文中就是指浏览器。

（5）`Authorization server`：认证服务器，即服务提供商专门用来处理认证的服务器。

（6）`Resource server`：资源服务器，即服务提供商存放用户生成的资源的服务器。它与认证服务器，可以是同一台服务器，也可以是不同的服务器。

知道了上面这些名词，就不难理解，OAuth的作用就是让"客户端"安全可控地获取"用户"的授权，与"服务商提供商"进行互动。

## 2. 四种模式区别

`授权码模式`没什么好讲的，应用最广泛，是标准的OAuth2模式；

`简化模式`适合没有后端的应用，也就无法安全的存储`client id` 和 `client secret` 例如只有一个页面的h5调查问卷等等；

`密码模式`是用户把账号密码直接告诉客户端，非常不安全，适用于遗留项目升级为oauth2的适配方案、客户端是自家的应用；

`客户端模式`直接根据client的id和密钥即可获取token，无需用户参与，适用于后端服务调后端服务；

### 参考链接：

[理解OAuth 2.0](https://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)

[oauth2四种授权方式小结](https://segmentfault.com/a/1190000012332319)

[[简易图解\]『 OAuth2.0』 『进阶』 授权模式总结](https://learnku.com/articles/20082)

[OAuth2.0的四种授权模式](https://www.cnblogs.com/alittlesmile/p/11531577.html)

