---
title: "dockerfile-maven-plugin使用"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"Maven"
]
series : [
"Manual"
]
images : [

]
---

[comment]: <> (# dockerfile-maven-plugin使用)



```xml
<properties>
    <docker.image.prefix>anaham-docker.pkg.coding.net/cereshop/ceres</docker.image.prefix>
    <anaham-docker.username>用户名</anaham-docker.username>
    <anaham-docker.password>密码</anaham-docker.password>
</properties>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        <!-- docker打包插件 -->
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>dockerfile-maven-plugin</artifactId>
            <version>${dockerfile-maven-plugin.version}</version>
            <configuration>
                <username>${anaham-docker.username}</username>
                <password>${anaham-docker.password}</password>
                <repository>${docker.image.prefix}/${project.artifactId}</repository>
                <tag>${ceres.version}</tag>  <!-- 不指定tag默认为latest -->
                <buildArgs>
                    <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
                </buildArgs>
            </configuration>
        </plugin>
    </plugins>
</build>
```

1. 构建镜像

```shell
mvn clean package -Dmaven.计算机科学.skip=true dockerfile:build -Ddockerfile.tag=latest
```

因为上面pom.xml已经指定了tag，也可以直接使用：`mvn dockerfile:build` 会优先选择pom.xml配置的tag

2. 上传镜像

```shell
mvn dockerfile:push -Ddockerfile.username=[镜像仓库账号] -Ddockerfile.password=[镜像仓库密码]
```

因为上面pom.xml已经指定了username和password，也可以直接使用：`mvn dockerfile:push`
