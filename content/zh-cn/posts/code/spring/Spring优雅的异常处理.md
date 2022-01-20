---
title: "Spring优雅的异常处理"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"Spring"
]
series : [
"Manual"
]
images : [

]
---

[comment]: <> (# Spring优雅的异常处理)

SpringBoot的WEB异常捕获，如果是WEB项目的话，可以直接处理Controller中的异常。如果不是WEB项目的话，就需要使用AspectJ来做切面。
### 1. web项目

```java
package com.test.handler;

import lombok.extern.log4j.Log4j2;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;


@ControllerAdvice
@Log4j2
public class GlobalExceptionHandler {
    @ExceptionHandler(value = Exception.class)
    public String exception(Exception e, Model model){
        log.error("find exception:e={}",e.getMessage());
        model.addAttribute("mes",e.getMessage());
        return "pages/500";
    }
}
```

**参考链接：**

[SpringBootWEB项目和非Web项目的全局异常捕获](https://www.cnblogs.com/songxingzhu/p/9718670.html)

[SpringBoot 处理异常的几种常见姿势](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485568&idx=2&sn=c5ba880fd0c5d82e39531fa42cb036ac&chksm=cea2474bf9d5ce5dcbc6a5f6580198fdce4bc92ef577579183a729cb5d1430e4994720d59b34&token=1729829670&lang=zh_CN&scene=21#wechat_redirect)

[使用枚举简单封装一个优雅的 Spring Boot 全局异常处理！](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486379&idx=2&sn=48c29ae65b3ed874749f0803f0e4d90e&chksm=cea24460f9d5cd769ed53ad7e17c97a7963a89f5350e370be633db0ae8d783c3a3dbd58c70f8&scene=178&cur_album_id=1322577180722872320#rd)

### 2. 非web项目

```java
package com.test.syncbackend.handler;

import lombok.extern.log4j.Log4j2;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

@Component
@Aspect
@Log4j2
public class GlobalExceptionHandler {

    @Pointcut("execution(* com.test.syncbackend.scheduleds.*.*(..))")
    public void pointCut() {
    }

    @Around("pointCut()")
    public Object handlerException(ProceedingJoinPoint proceedingJoinPoint) {
        try {
            return proceedingJoinPoint.proceed();
        } catch (Throwable ex) {
            log.error("execute scheduled occur exception.", ex);
        }
        return null;
    }
}
```

