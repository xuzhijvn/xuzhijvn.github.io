---
title: "Centos Linux清理磁盘空间"
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

]
---

Centos Linux清理磁盘空间



1. `df -hl` 查看占比

 

```shell
[root@VM-0-12-centos apioak-document]# df -hl
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        1.9G     0  1.9G   0% /dev
tmpfs           1.9G   24K  1.9G   1% /dev/shm
tmpfs           1.9G  788K  1.9G   1% /run
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/vda1        50G   48G     0 100% /
tmpfs           379M     0  379M   0% /run/user/0
overlay          50G   48G     0 100% /var/lib/docker/overlay2/b3b7c9bcd6a086d488f5ca533ec7fc934f863340e4efa40513c354f3a13c6ccd/merged
shm              64M     0   64M   0% /var/lib/docker/containers/76627689f39c84201d3554e95cf7a8ca53bc53f660fbf1127d86c49db2a67591/mounts/shm
overlay          50G   48G     0 100% /var/lib/docker/overlay2/f3fac618fb01637e586b483c87d9075d1cb08bee9c9d1f3afffd0a012394b067/merged
shm              64M     0   64M   0% /var/lib/docker/containers/36172ef8f5a337d595f41fda4a2e7864b333278a9d2102b700d7f2a6f47c52a8/mounts/shm
```

2. 在根目录执行`du -sh *`

```shell
[root@VM-0-12-centos /]# du -sh *
0	bin
147M	boot
987M	data
0	dev
6.5G	docker
39M	etc
4.0K	home
0	lib
0	lib64
16K	lost+found
4.0K	media
4.0K	mnt
20K	opt
du: cannot access ‘proc/31203/task/31203/fd/4’: No such file or directory
du: cannot access ‘proc/31203/task/31203/fdinfo/4’: No such file or directory
du: cannot access ‘proc/31203/fd/4’: No such file or directory
du: cannot access ‘proc/31203/fdinfo/4’: No such file or directory
0	proc
1.6G	root
788K	run
0	sbin
4.0K	srv
0	sys
181M	tmp
7.4G	usr
31G	var
```

3. cd到空间占用大的目录继续执行`du -sh *`

```shell
[root@VM-0-12-centos var]# du -sh *
4.0K	adm
131M	cache
4.0K	crash
20K	db
8.0K	empty
4.0K	games
4.0K	gopher
12K	kerberos
27G	lib
4.0K	local
0	lock
4.5G	log
0	mail
4.0K	nis
4.0K	opt
4.0K	preserve
0	run
124K	spool
24K	tmp
4.0K	yp
```

清除占用大的log

参考：

[Centos Linux 怎么清理磁盘占用空间大：/dev/xvda1](https://blog.csdn.net/cen_cs/article/details/54861704)
