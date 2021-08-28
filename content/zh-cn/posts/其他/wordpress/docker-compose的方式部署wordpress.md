---
title: "docker-compose的方式部署wordpress"
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

docker-compose的方式部署wordpress

- 安装docker

```shell
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install docker-ce docker-ce-cli containerd.io
```

- 编写docker-compose.yml

```yaml
mysql:
    image: mysql:5.7
    environment:
     - MYSQL_ROOT_PASSWORD=123456
     - MYSQL_DATABASE=wordpress
web:
    image: wordpress
    links:
     - mysql
    environment:
     - WORDPRESS_DB_PASSWORD=123456
    ports:
     - "0.0.0.0:8080:80"
    working_dir: /var/www/html
    volumes:
     - wordpress:/var/www/html
```

- 启动docker

```
sudo systemctl start docker
```

- 运行docker-compose

```shell
docker-compose up
```

参考：

[What does "local address" and "foreign address" mean in the netstat command result?](https://www.quora.com/What-does-local-address-and-foreign-address-mean-in-the-netstat-command-result)

[Docker 入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)

[Docker 微服务教程](http://www.ruanyifeng.com/blog/2018/02/docker-wordpress-tutorial.html)

[Wordpress 容器化、HTTPS化全攻略](https://netsecurity.51cto.com/art/201906/598459.htm)
