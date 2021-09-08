---
title: "WordPress改造成https的注意事项"
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
"images/center.png"
]
---

WordPress改造成https的注意事项



WordPress加入ssl后可能出现的网站访问缓慢、样式无法被加载，是由于站点虽然被改造成了https访问，但是 WordPress 代码层面对于一些css、js、图片等静态资源的访问还是http的，所以才会出现这种情况。解决的办法可以改造代码，也可以安装WordPress插件。下面介绍后者的步骤：



1、申请证书、上传证书到服务器、配置服务器（阿里云有免费的证书，并有详细的操作步骤）



2、安装 **Really Simple SSL**插件，它会将http的请求全都转成https（感谢作者吧）



3、 后台修改wordpress地址和站点地址，如下图所示：



![img](https://picgo.6and.ltd/img/https-setup.png)



参考链接：https://www.rogoso.info/wordpress-ssl/
