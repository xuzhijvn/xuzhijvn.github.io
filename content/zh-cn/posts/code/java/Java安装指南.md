---
title: "Java安装指南"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"Java"
]
series : [
"Manual"
]
images : [

]
---

[comment]: <> (# Java安装指南)

## 1. 下载JDK

进入[Oracle 官方网站](https://www.oracle.com/java/technologies/javase-jdk8-downloads.html)下载合适的 JDK 版本，准备安装。

## 2. 创建目录

执行如下命令，在 /usr/ 目录下创建 java 目录。

```shell
mkdir /usr/java
cd /usr/java
```

 

将下载的文件 jdk-8u151-linux-x64.tar.gz 复制到 /usr/java/ 目录下。

## 3. 解压 JDK

```shell
tar -zxvf jdk-8u151-linux-x64.tar.gz
```

## 4. 设置环境变量

```shell
set java environment
JAVA_HOME=/usr/java/jdk1.8.0_151        
JRE_HOME=/usr/java/jdk1.8.0_151/jre     
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
```

使修改生效：

```shell
source /etc/profile
```

## 5. 测试

执行如下命令进行测试。

```shell
java -version
```

若显示 Java 版本信息，则说明 JDK 安装成功：

```shell
java version "1.8.0_151"
Java(TM) SE Runtime Environment (build 1.8.0_151-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.151-b12, mixed mode)
```

