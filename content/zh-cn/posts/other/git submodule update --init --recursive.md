---
title: "git submodule update --init --recursive"
date: 2021-08-28T11:15:10+08:00
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

执行 `git submodule update --init --recursive` 的时候报错：

```sh
fatal: remote error: upload-pack: not our ref fc7223ca00124e8f5b5b354457379071e2fd091b
Fetched in submodule path 'themes/hugo-theme-bootstrap', but it did not contain fc7223ca00124e8f5b5b354457379071e2fd091b. Direct fetching of that commit failed.
```

解决：

```sh
cd {submodule path}
git reset --hard origin/master
cd -
git clean -n
git add {submodule path}
git commit
git submodule update --init --recursive
```

## 参考

[git - 为什么git子模块更新会因 "fatal: remote error: upload-pack: not our ref"而失败？](https://www.coder.work/article/7542928)

