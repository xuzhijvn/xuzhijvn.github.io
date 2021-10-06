+++
title = "Note"
description = ""
date = 2021-10-03T11:20:44+08:00
featured = false
draft = true
comment = true
toc = true
reward = true
categories = [
  ""
]
tags = [
  ""
]
series = []
images = []
+++

<!--more-->

##### hashmap扩容机制

当map中包含的Entry的数量大于等于threshold = loadFactor * capacity的时候，且新建的Entry刚好落在一个非空的桶上，此刻触发扩容机制，将其容量扩大为2倍。调用resize扩容。

##### 印象深刻的经历

一次缓存优化经历，

##### 为什么跳槽？

##### go channel

https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/

##### netty面试

##### spring单例vs多例

Spring Bean有五种作用域：

`singleton` ,  `prototype`,  `request` , `session` ,  `global session`

默认是单例，可以通过xml配置bean多例，更方便的是用@Scope("prototype")注解。

Controller, Service, DAO默认都是单例的。

单例是线程不安全的，所以不要在类中使用状态可变的成员变量。

##### @Autowired 和 @Resourse区别

两者都可以写在字段和setter方法上

@Autowired是Spring中定义注解，@Resourse是JRE定义的注解

@Autowired默认按byType装配，需要按byName来装配，可以结合@Qualifier注解一起使用，@Resourse默认按byName装配，它有可以配置的属性 name和type

```java
public class TestServiceImpl {
    @Autowired
    @Qualifier("userDao")
    private UserDao userDao; 
}
```



##### tcp三握四分 + SpringMVC 打开网站看

##### Spring设计模式

适配器 + 工厂 + 代理 + 单例

##### 数据库深分页优化



3个问题，技术栈是什么？+ 团队负责的工作内容是？+  今天的表现做怎么样的评价？
