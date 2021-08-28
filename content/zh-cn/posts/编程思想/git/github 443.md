---
title: "GitHub 443"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"github"
]
series : [
"Manual"
]
images : [
"images/center.png"
]
---

[comment]: <> (# GitHub 443)

我们真是一个神奇的国度，连github都要封禁。

最近github https无法接入：

```shell
fatal: unable to access 'https://github.com/xuzhijvn/tony-demo.git/': LibreSSL SSL_connect: SSL_ERROR_SYSCALL in connection to github.com:443 
```

试了n种方式都不行：

1. 切换SSR代理节点

2. 切换SSR代理模式到全局

3. 退出SSR

4. 使用蓝灯代理

5. 设置git代理（61885是我蓝灯代理的端口）

```shell
git config --global --list

git config --global https.proxy '127.0.0.1:61885'
git config --global http.proxy '127.0.0.1:61885'

git config --global --list
```

6. 禁用git代理

```shell
git config --global --list

git config --global --unset http.proxy

git config --global --list
```

7.  `networksetup -setv6off Wi-Fi` 

以上方法全部不好使，折腾了一下午心态崩了💔💔

最后只能改成通过ssh方式接入了：

1. 生成公私钥对 

```shell
ssh-keygen -t rsa -C "783175223@qq.com"
```

2. 拷贝到github

```shell
pbcopy < ~/.ssh/github_id_rsa.pub
```

3. 要使用`ssh-add`命令是把专用密钥添加到`ssh-agent`的高速缓存中

```shell
ssh-add -K ~/.ssh/github_id_rsa
```

4. 上面的命令在重启电脑之后会失效，所以得通过在`~/.ssh/config`文件中添加如下内容来取代：

```shell
Host *
AddKeysToAgent yes
UseKeychain yes
IdentityFile ~/.ssh/github_id_rsa
IdentityFile ~/.ssh/huolala_id_rsa
```

5. 接入github的时候用ssh模式

# 参考链接：

[工作、开源两不误：Git多账号管理](https://zhuanlan.zhihu.com/p/62071906)
