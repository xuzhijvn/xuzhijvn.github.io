---
title: "Jenkins+docker+腾讯云容器仓库+github搭建CI/CD环境"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"持续集成部署"
]
tags : [
"Jenkins",
"docker"
]
images : [
"images/center.png"
]
---

持续集成/持续部署的重要性不必多言，都什么年代了，没有哪个正紧项目还在人工构建/测试/部署。本文手把手教你搭建Jenkins+docker+腾讯云容器仓库+github的CI/CD环境。妥妥的干活，绝对是解放生产力的利器。架构图如下：

<img src="https://picgo.6and.ltd/img/20200510052839100.png" alt="jenkins+docker+腾讯云容器仓库+github搭建CI/CD环境" style="zoom: 43%;" />

step1: 程序猿git push代码到github
step2: jenkins通过github webhooks触发任务（ jenkins构建项目，并制作docker镜像）
step3: 推送镜像到容器仓库
step4: 应用服务器拉去镜像到本地运行

为了不让文章篇幅过长，本系列tony老师将文章分成两个小章节：

## Github webhooks+Jenkins搭建持续集成环境

第一章节，意在教您通过github webhooks+jenkins搭建持续集成环境，如果您已经完成webhooks+jenkins的配置，可以直接调整到第二章节

<img src="https://picgo.6and.ltd/img/2020051008240188.png" alt="2020051008240188" style="zoom: 45%;" />

### 1. github配置Personal access tokens

没什么好说的，按图操作：

<img src="https://picgo.6and.ltd/img/2020051008301081.png" alt="2020051008301081"  />





<img src="https://picgo.6and.ltd/img/2020051008320846.png" alt="2020051008320846"  />



记下生成的token，待会儿要用的。

### 2. github配置webhooks

去到你的github项目仓库，按图操作：

![2020051008353493](https://picgo.6and.ltd/img/2020051008353493.png)



![2020051008384728](https://picgo.6and.ltd/img/2020051008384728.png)

这里的Payload URL = 你的jenkins地址+/github-webhook/

至此，github上的配置全部完成。接下来，我们配置jenkins。

------

### 3. jenkins配置凭据

![2020051008473068](https://picgo.6and.ltd/img/2020051008473068.png)



![2020051008483017](https://picgo.6and.ltd/img/2020051008483017.png)



这里的Secret就是在第1节配置的Personal access tokens



![2020051008512188](https://picgo.6and.ltd/img/2020051008512188.png)



1. 添加一个github登陆凭据，username和password对应github的登陆账号密码。
2. 添加一个宿主机登陆凭据，username和password对应登陆的账号密码（因为小编的jenkins是容器的方式运行的，因此后面需要这个凭据ssh到宿主机上执行命令）。

### 4. 配置ssh sites

接下来配置一个ssh sites，用于后面需要ssh到宿主机。

![2020051009070887](https://picgo.6and.ltd/img/2020051009070887.png)

### 5. 配置jenkins运行环境

这里的环境取决与你的项目需要的环境。

![2020051009254533](https://picgo.6and.ltd/img/2020051009254533.png)

如果你的jenkins是直接运行在机器上，那么指定对应环境的路径即可。如果是运行在容器里面，容器启动的时候把宿主机的环境映射到容器即可。

### 6. 新建一个maven项目

本小节以一个maven项目为例子，下一个小节我还补充了vue项目的例子。

<img src="https://picgo.6and.ltd/img/202005100909402.png" alt="img"  />
请按图操作：

<img src="https://picgo.6and.ltd/img/2020051009372456.png" alt="img"  />

请按图操作：

<img src="https://picgo.6and.ltd/img/2020051009154947.png" alt="img"  />

请按图操作：

<img src="https://picgo.6and.ltd/img/2020051009181624.png" alt="img"  />

这里对上图解释一下：上图的含义是，当maven构建完项目之后，制作docker镜像，并推送到远程容器仓库。当然，你也可以构建完项目之后，直接java -jar本地运行，这一切取决于你的需求！！！

### 7. 新建一个vue项目

你的前端可能是通过vue实现前后端分离的，那么你肯定也存在对vue编译构建的需求。

<img src="https://picgo.6and.ltd/img/2020051009305674.png" alt="img"  />

这里大部分配置和第5小节是一样的，不再赘述，需要注意的是：

<img src="https://picgo.6and.ltd/img/2020051009404851.png" alt="img"  />

上图中command的含义是: 1. 构建项目 2. 运行项目 3. 制作镜像 4，推送镜像到仓库。同第5节的maven项目一样，需要哪些步骤取决于你的需求。

通过本文，您已经掌握了当开发人员提交代码到github之后，jenkins自动构建项目/运行项目的持续集成方法。

本系列文章的[jenkins+腾讯云容器仓库+docker持续部署](https://xuzhijun.ltd/archives/677)将教你将构建好的项目制作成docker镜像，并由真正的应用服务器去运行项目，真正实现CI/CD，解放生产力。

## Jenkins+腾讯云容器仓库+docker持续部署

第二章节，意在教您通过jenkins+腾讯云容器仓库+docker搭建持续部署环境，如果您还没有完成webhooks+jenkins的配置，请完成第一章节的学习

<img src="https://picgo.6and.ltd/img/2020051008231993.png" alt="img" style="zoom:50%;" />

### 1. 准备

在上一个章节的基础上，您首先要：

1. jenkins添加您应用服务器的登陆凭证（参照第一章第3小节）
2. jenkins添加您应用服务器的ssh sites（参照第一章第4小节）
3. 开通腾讯云容器服务（免费开通[镜像仓库基本教程](https://cloud.tencent.com/document/product/457/9117)）
4. 本章的项目源码来自[yshop意象商城系统](https://gitee.com/guchengwuyue/yshopmall)

### 2. 项目增加docker配置

#### 2.1 h5项目新增docker配置

新增如下3个配置：构建镜像脚本build-image.sh、镜像定义Dockerfile-h5、推送到容器仓库脚本push.sh

![2020051010202528](https://picgo.6and.ltd/img/2020051010202528.png)

<!--文章末尾获取配置文件！-->

#### 2.2 后台管理前端项目新增docker配置

新增如下3个配置：

![2020051010334163](https://picgo.6and.ltd/img/2020051010334163.png)

<!--文章末尾获取配置文件！-->

#### 2.3 后台项目新增docker配置

新增如下10个配置：
<img src="https://picgo.6and.ltd/img/2020051010521993.png" alt="img"  />

<!--文章末尾获取配置文件！-->

### 3. 修改项目配置文件

修改mysql的地址为“db”,redis的地址为“redis”,mysql链接新增 allowPublicKeyRetrieval=true&useSSL=false参数

<img src="https://picgo.6and.ltd/img/2020051011013955.png" alt="img"  />



![2020051011035934](https://picgo.6and.ltd/img/2020051011035934.png)

### 4. jenkins配置

如果您已经完成了第一章中的配置，那您只需要新增一小部分配置即可。

#### 4.1 新增h5项目的Execute shell script on remote host using ssh

![202005101117138](https://picgo.6and.ltd/img/202005101117138.png)



1. jenkins宿主机构建镜像并push到容器仓库
2. ssh到应用服务器下载镜像并启动项目

#### 4.2 新增前端项目的Execute shell script on remote host using ssh

![img](https://picgo.6and.ltd/img/2020051011214721.png)

1. jenkins宿主机构建镜像并push到容器仓库
2.  ssh到应用服务器下载镜像并启动项目

#### 4.3 新增后台项目的Execute shell script on remote host using ssh

![img](https://picgo.6and.ltd/img/2020051011255759.png)

1. jenkins宿主机构建镜像并push到容器仓库
2. ssh到应用服务器下载镜像并启动项目

### 5. 启动

#### 5.1 修改docker-compose-env.yml中mysql的密码

#### 5.2 启动项目

```shell
#启动运行环境
docker-compose -f docker-compose-env.yml up &

#建数据库
cp ~/repository/yshop/sql/yshop2.0.sql /data/mysql/log
docker exec -it mysql bash
cd /var/log/mysql
mysql -uroot -proot
create database yshop;
use yshop;
source yshop2.0.sql
exit;

#启动 system/api/h5/admin
docker-compose -f docker-compose-app.yml up &
```

#### 5.3 nginx配置

将app.conf文件放到/data/nginx/vhost目录

```shell
docker-compose restart -f docker-compose-env.yml nginx
```

> 注意⚠️：
>
> 需要替换server_name成你自己域名
> 项目源码中的api默认是https的，如果你没有ssl证书，或者嫌麻烦，请自行改成http。

![img](https://picgo.6and.ltd/img/img_5eb7f8bbad443-20210604143620981.png)
![img](https://picgo.6and.ltd/img/img_5eb7f8db1dd68-20210604143629235.png)

至此，您已经完成全部配置，提交代码即可自动完成构建/部署/运行，当然你也可以在jenkins上手动点击构建。

### 6. 配置文件

配置文件下载：

https://pan.baidu.com/s/1KQT3vRgYXby3FHPViejzhQ

提取码：hrqs

[![img](https://picgo.6and.ltd/img/2020051011572178.png)](https://xuzhijun.ltd/wp-content/uploads/2020/05/2020051011572178.png)