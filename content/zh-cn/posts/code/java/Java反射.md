---
title: "Java反射"
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

[comment]: <> (# Java反射)

反射可以在程序运行过程中动态的构造类、获取类的全部信息、调用类型方法。但是，为什么我们要这么做呢？需要构造类，new就好了，需要访问类成员变量、调用方法，直接访问、调用就好了，为什么要通过一大堆反射代码去实现呢？

通常，class在编译期间就确定，JVM在运行时通过类加载器加载确定的class。如果在运行时才确定需要加载什么类，就需要利用java反射。java反射使得程序更加灵活，类似spring的框架将类以全限定名的形成配置在配置文件，然后再通过反射实例化。

参考：

https://blog.csdn.net/Appleyk/article/details/77879073
