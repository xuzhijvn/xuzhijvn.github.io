---
title: "为docker容器安装工具"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"持续集成部署"
]
tags : [
"docker"
]
images : [

]
---

为docker容器安装工具

因为我的docker jenkins需要node环境，默认是没有的，所以最初想法是和maven一样，将宿主机的maven和docker一起共享使用，但是不管怎么操作就不行，很诡异，同样的操作，maven可以执行，jdk和node却不行，如下所示：

```yaml
version: '3'
services:
  jenkins:
    image: jenkinsci/blueocean
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 1024M
        reservations:
          memory: 64M
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
      - '/usr/bin/docker:/usr/bin/docker'
      - '/usr/local/node-v12.16.3-linux-x64:/usr/local/node-v12.16.3-linux-x64'
    restart: always
```

![img](https://cdn.tkaid.com/img/img_5eb4d21387fdc-20210621143038464.png)

![img](https://cdn.tkaid.com/img/img_5eb4d1e12de8e-20210621143042626.png)

> 更诡异的是，虽然docker下无法执行node命令，jenkins却能调用：
>
> ![img](https://cdn.tkaid.com/img/img_5eb4d29cd4624-20210621143052299.png)
>
> ![img](https://cdn.tkaid.com/img/img_5eb4d31ee9ef8-20210621143432171.png)
>
> 这里暂且不管，请当作docker没有node环境，继续往下阅读。

经过彻夜未眠的排除，基本能定位为题的原因：本质不是node命令找不到，而是node的依赖在docker容器里面找不到，node依赖如下：

![img](https://cdn.tkaid.com/img/img_5eb4c4003270b-20210621143427506.png)

所以要么进入到容器里面使用apk add，要么自己制作镜像（这样太麻烦了），我选择前者。

进入到容器里面安装：

```shell
apk add --no-cache nodejs
```

> 可能会遇到下载缓慢、找不到包等问题。

解决办法如下：

> 在 https://mirrors.alpinelinux.org/ 找到中国的镜像仓库，并添加到 etc/apk/repositories, 参考如下：
> A. 永久修改下载源
>
> ```shell
> vi etc/apk/repositories
> http://mirrors.aliyun.com/alpine/v3.8/main/
> http://mirrors.aliyun.com/alpine/v3.8/community/
> apk update
> ```
>
> B. 临时修改下载源地址
> 如果上面的永久源地址找不到对应的包，可使用本方法，参考如下：
> B.1. 先通过 https://pkgs.alpinelinux.org/packages 找到你要下载的package对应的Branch和Repository。
> B.2. 添加package
>
> ```shell
> apk add gradle --repository http://mirrors.tuna.tsinghua.edu.cn/alpine/edge/community
> ```

参考链接：
https://pkgs.alpinelinux.org/packages
https://mirrors.alpinelinux.org/
https://www.cnblogs.com/sunsky303/p/11548343.html
https://my.oschina.net/u/1422143/blog/1858790
