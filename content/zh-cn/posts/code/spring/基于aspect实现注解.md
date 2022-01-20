---
title: "基于aspect实现注解"
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

[comment]: <> (# 基于aspect实现注解)

在工作中，我们有时候需要将一些公共的功能封装，比如操作日志的存储，防重复提交等等。这些功能有些接口会用到，为了便于其他接口和方法的使用，做成自定义注解，侵入性更低一点。别人用的话直接注解就好。下面就来讲讲自定义注解这些事情。

## 1. @Target、@Retention、@Documented简介

java自定义注解的注解位于包：java.lang.annotation下。包含三个元注解@Target、@Retention、@Documented，即注解的注解。

### @Target

`@Target`:注解的作用目标。和枚举ElementType共同起作用

根据源码知道，可以配置多个作用目标。

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    /**
     * Returns an array of the kinds of elements an annotation type
     * can be applied to.
     * @return an array of the kinds of elements an annotation type
     * can be applied to
     */
    ElementType[] value();
}
复制代码
```

ElementType的类型如下：

```java
* @author  Joshua Bloch
 * @since 1.5
 * @jls 9.6.4.1 @Target
 * @jls 4.1 The Kinds of Types and Values
 */
public enum ElementType {
    /** 类, 接口 (包括注解类型), 或 枚举 声明 */
    TYPE,
 
    /** 字段声明（包括枚举常量） */
    FIELD,
 
    /** 方法声明(Method declaration) */
    METHOD,
 
    /** 正式的参数声明 */
    PARAMETER,
 
    /** 构造函数声明 */
    CONSTRUCTOR,
 
    /** 局部变量声明 */
    LOCAL_VARIABLE,
 
    /** 注解类型声明 */
    ANNOTATION_TYPE,
 
    /** 包声明 */
    PACKAGE,
 
    /**
     * 类型参数声明
     *
     * @since 1.8
     */
    TYPE_PARAMETER,
 
    /**
     * 使用的类型
     *
     * @since 1.8
     */
    TYPE_USE
}
复制代码
```

### @Retention

指示带注解类型的注解要多长时间被保留。如果没有保留注释存在 注释类型声明，保留策略默认为  {@code RetentionPolicy.CLASS}和RetentionPolicy共同起作用。

RetentionPolicy类型如下：

```java
package java.lang.annotation;

/**
 * Annotation retention policy.  The constants of this enumerated type
 * describe the various policies for retaining annotations.  They are used
 * in conjunction with the {@link Retention} meta-annotation type to specify
 * how long annotations are to be retained.
 *
 * @author  Joshua Bloch
 * @since 1.5
 */
public enum RetentionPolicy {
    /**
     * 注解只保留在源文件，当Java文件编译成class文件的时候，注解被遗弃；被编译器忽略
     */
    SOURCE,

    /**
     * 注解被保留到class文件，但jvm加载class文件时候被遗弃，这是默认的生命周期
     */
    CLASS,

    /**
     * 注解不仅被保存到class文件中，jvm加载class文件之后，仍然存在
     */
    RUNTIME
}

复制代码
```

这3个生命周期分别对应于：Java源文件(.java文件) ---> .class文件 ---> 内存中的字节码。

> 那怎么来选择合适的注解生命周期呢？
>
> 首先要明确生命周期长度 **SOURCE < CLASS < RUNTIME** ，所以前者能作用的地方后者一定也能作用。一般如果需要**在运行时去动态获取注解信息，那只能用 RUNTIME 注解**；如果要**在编译时进行一些预处理操作**，比如生成一些辅助代码（如 [ButterKnife](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FJakeWharton%2Fbutterknife)）**，就用 CLASS注解**；如果**只是做一些检查性的操作**，比如 **@Override** 和 **@SuppressWarnings**，则**可选用 SOURCE 注解**。

### **@**Documented

**@**Documented 注解表明这个注解应该被 javadoc工具记录. 默认情况下,javadoc是不包括注解的. 但如果声明注解时指定了 @Documented,则它会被 javadoc 之类的工具处理, 所以注解类型信息也会被包括在生成的文档中，是一个标记注解，没有成员。

## 2. 实战

Log注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Log {
    String value() default "";
}
```

Aspect

```java
@Aspect
@Component
@Slf4j
public class SysLogAspect {

    @Pointcut("@annotation(com.tony.annotation.aspect.Log)")
    public void logPointCut() {
    }

    @Around("logPointCut()")
    public Object around(ProceedingJoinPoint point) throws Throwable {
        long beginTime = System.currentTimeMillis();
        
        Object result = point.proceed();
        long time = System.currentTimeMillis() - beginTime;
        
        // 目标方法
        MethodSignature signature = (MethodSignature) point.getSignature();
        Method method = signature.getMethod();

        Log syslog = method.getAnnotation(Log.class);
        if (syslog != null) {
            // 注解上的描述
            log.info(syslog.value() + " :" + time);
        }
        return result;
    }
}
```

使用

```java
@Service
public class HelloLog {

    @Log("测试自定义注解")
    public void sayHello() {
        System.out.println("hello, log" );
    }
}
```

测试

```java
@RunWith(SpringRunner.class)
@SpringBootTest
class AnnotationApplicationTests {

    @Autowired
    HelloLog helloLog;
    
    @Test
    public void HelloLogTest(){
        helloLog.sayHello();
    }
}
```



## 参考

[分分钟玩转SpringBoot自定义注解](https://juejin.cn/post/6894912314894450702)
