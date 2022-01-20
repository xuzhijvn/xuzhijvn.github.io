---
title: "docker-compose的方式启动jenkins"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"持续集成部署"
]
tags : [
"Jenkins",
"docker-compose"
]
images : [

]
---

docker-compose的方式启动jenkins

jenkins官网提供了多种安装方式，唯独没有提供docker compose的教程！咱也不知道为啥，所以自己动手丰衣足食。

## 1. 编写docker-compose.yml

```yaml
version: '3'
services:
  jenkins:
    image: jenkinsci/blueocean
    container_name: jenkins
    user: root
    ports:
      - '8080:8080'
      - '50000:50000'
    volumes:
      - '/docker/volumes/jenkins:/var/jenkins_home'
      - '/var/run/docker.sock:/var/run/docker.sock'
      - '/usr/local/jdk1.8.0_241:/usr/local/jdk1.8.0_241'
      - '/usr/local/apache-maven-3.6.3:/usr/local/apache-maven-3.6.3'
    restart: always
```

## 2. 启动jenkins

```shell
docker-compose up -d jenkins && docker logs -f jenkins
```

==jenkins 启动时，一直处在 Please wait while Jenkins is getting ready to work ...==
需要更新hudson.model.UpdateCenter.xml的url：

```xml
<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
    <url>https://updates.jenkins.io/update-center.json</url>
  </site>
</sites>
```

更新为：

```xml
<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
    <url>http://mirror.xmission.com/jenkins/updates/update-center.json</url>
  </site>
</sites>
```
