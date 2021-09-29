+++
title = "SpringAOP执行顺序"
description = ""
date = 2021-08-31T14:50:56+08:00
featured = false
comment = true
toc = true
reward = true
categories = [
  "编程思想"
]
tags = [
"Spring"
]
series = [
"Manual"
]
images = [
"images/center.png"
]

+++

<!--more-->

### 1. 一个方法只被一个Aspect类拦截

在一个方法只被一个aspect类拦截时，aspect类内部的 advice 将按照以下的顺序进行执行：

**正常情况：** 
<img src="https://picgo.6and.ltd/img/20160811192425854-20210831150318603.png" alt="one-ok" style="zoom: 67%;" />

 

------

 

**异常情况：** 
<img src="https://picgo.6and.ltd/img/20160811192446479-20210831150323983.png" alt="one-exception" style="zoom:67%;" />

### 2. 同一个方法被多个Aspect类拦截

比如，我们为 apsect1 和 aspect2 分别添加 @Order 注解，如下：

```java
@Order(5)
@Component
@Aspect
public class Aspect1 {
    // ...
}
 
@Order(6)
@Component
@Aspect
public class Aspect2 {
    // ...
}
```

这样修改之后，可保证不管在任何情况下， aspect1 中的 advice 总是比 aspect2 中的 advice 先执行。如下图所示： 
<img src="https://picgo.6and.ltd/img/20160811193349342.png" alt="two-ok" style="zoom:67%;" />

### 注意点

- 如果在同一个 aspect 类中，针对同一个 pointcut，定义了两个相同的 advice(比如，定义了两个 @Before)，那么这两个 advice 的执行顺序是无法确定的，哪怕你给这两个 advice 添加了 @Order 这个注解，也不行。这点切记。
- 对于`@Around`这个advice，不管它有没有返回值，但是必须要方法内部，调用一下 `pjp.proceed();`否则，Controller 中的接口将没有机会被执行，从而也导致了 `@Before`这个advice不会被触发。比如，我们假设正常情况下，执行顺序为”aspect2 -> apsect1 -> controller”，如果，我们把 `aspect1`中的`@Around`中的 `pjp.proceed();`给删掉，那么，我们看到的输出结果将是：

```java
[Aspect2] around advise 1
[Aspect2] before advise
[Aspect1] around advise 1
[Aspect1] around advise2
[Aspect1] after advise
[Aspect1] afterReturning advise
[Aspect2] around advise2
[Aspect2] after advise
[Aspect2] afterReturning advise
```

从结果可以发现， Controller 中的 接口 未被执行，aspect1 中的 `@Before` advice 也未被执行。

## 参考

[Spring Aop实例@Aspect、@Before、@AfterReturning@Around 注解方式配置](https://blog.csdn.net/u012843873/article/details/80540499)

