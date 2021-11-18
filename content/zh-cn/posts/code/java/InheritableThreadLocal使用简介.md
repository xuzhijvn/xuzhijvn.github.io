+++
title = "InheritableThreadLocal使用简介"
description = ""
date = 2021-11-18T14:51:43+08:00
featured = false
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

在做日志链路追踪的场景中，我们需要将traceId从父线程传递到子线程，我们无法直接通过ThreadLocal进行值传递：

```java
public class Main {
    
    private static ThreadLocal<String> threadLocal = new ThreadLocal<>();
    
    public static void main(String[] args) {
        threadLocal.set("mainThread");
        System.out.println("value:"+threadLocal.get());
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                String value = threadLocal.get();
                System.out.println("value:"+value);
            }
        });
        thread.start();
    }
}
```

输出：

```java
value:mainThread
value:null
```

InheritableThreadLocal继承自ThreadLocal，在Thread的init方法中有这样的逻辑：

```java
        if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
```

即，线程的inheritThreadLocals是会从父线程传递到子线程的：

```java
public class Main {
    
    private static InheritableThreadLocal<String> threadLocal = new InheritableThreadLocal<>();
    
    public static void main(String[] args) {
        threadLocal.set("mainThread");
        System.out.println("value:"+threadLocal.get());
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                String value = threadLocal.get();
                System.out.println("value:"+value);
            }
        });
        thread.start();
    }
}
```

输出：

```
value:mainThread
value:mainThread
```



我们在过滤器中设置traceId：

```java
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;

        // 从请求头中获取traceId
        String traceId = request.getHeader("traceId");
        // 不存在就生成一个
        if (traceId == null || "".equals(traceId)) {
            traceId = UUID.randomUUID().toString();
        }
        // 放入MDC中
        MDC.put("traceId", traceId);
        chain.doFilter(servletRequest, servletResponse);
    }
```

SpringBoot中默认使用的日志框架是logback，Slf4j提供了MDC ，具体的装饰者实现类是LogbackMDCAdapter

在早期的版本中子线程可以直接自动继承父线程的MDC容器中的内容，因为MDC在早期版本中使用的是InheritableThreadLocal来作为底层实现。但是由于性能问题被取消了，最后还是使用的是ThreadLocal来作为底层实现。这样子线程就不能直接继承父线程的MDC容器。

```java
package ch.qos.logback.classic.util;

import java.util.Collections;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

import org.slf4j.spi.MDCAdapter;

public class LogbackMDCAdapter implements MDCAdapter {

    final ThreadLocal<Map<String, String>> copyOnThreadLocal = new ThreadLocal<Map<String, String>>();

    private static final int WRITE_OPERATION = 1;
    private static final int MAP_COPY_OPERATION = 2;

    // keeps track of the last operation performed
    final ThreadLocal<Integer> lastOperation = new ThreadLocal<Integer>();

    private Integer getAndSetLastOperation(int op) {
        Integer lastOp = lastOperation.get();
        lastOperation.set(op);
        return lastOp;
    }
```

所以，Logback官方建议我们在父线程新建子线程之前调用MDC.getCopyOfContextMap()方法将MDC内容取出来传给子线程，子线程在执行操作前先调用MDC.setContextMap()方法将父线程的MDC内容设置到子线程。



```java
    @Override
    public void execute(Runnable command) {
        // 提交者的本地变量
        Map<String, String> contextMap = MDC.getCopyOfContextMap();
        super.execute(()->{
            if (contextMap != null) {
                // 如果提交者有本地变量，任务执行之前放入当前任务所在的线程的本地变量中
                MDC.setContextMap(contextMap);
            }
            try {
                command.run();
            } finally {
                // 任务执行完，清除本地变量，以防对后续任务有影响
                MDC.clear();
            }
        });
    }
```



## 参考

[Slf4j MDC机制](https://www.jianshu.com/p/1dea7479eb07)

[【并发编程】InheritableThreadLocal使用详解](https://www.cnblogs.com/54chensongxia/p/12015443.html)

[分布式系统中如何优雅地追踪日志（原理篇）](https://cloud.tencent.com/developer/article/1580354)

