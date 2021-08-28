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

navicat远程连接mysql，2003 can't connect to mysql server on 10038



首先我们通过

①：netstat -an|grep 3306

来查看mysql默认的端口3306是否开启，允许哪个ip使用，如果你发现，前面有127.0.0.1，就说明，3306端口只能本机ip使用

所以，我们需要

②：打开mysql配置文件vi /etc/mysql/mysql.conf.d/mysqld.cnf

将bind-address = 127.0.0.1注销

③：进入mysql，对远程用户进行授权，

grant all privileges on *.* to 'root'@'%' identified by 'xxxxxx';

这里的root 是你远程登录的用户，xxxxxx是你登录使用的密码，然后可以在mysql数据 表中查看到你这个用户已经被添加到user表中

④：service mysql restart
