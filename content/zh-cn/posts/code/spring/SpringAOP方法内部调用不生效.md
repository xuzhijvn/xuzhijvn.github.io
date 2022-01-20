---
title: "SpringAOP方法内部调用不生效"
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


[comment]: <> (# SpringAOP方法内部调用不生效)

假设一个接口里面有两个方法：

```java
package demo.long;

public interface CustomerService {  
    public void doSomething1();  
    public void doSomething2();  
}  
```

接口实现类如下：

```java
package demo.long.impl;

import demo.long.CustomerService; 

public class CustomerServiceImpl implements CustomerService {  
  
    public void doSomething1() {  
        System.out.println("CustomerServiceImpl.doSomething1()");  
        doSomething2();  
    }  
  
    public void doSomething2() {  
        System.out.println("CustomerServiceImpl.doSomething2()");  
    }  
  
}  
```

现在我需要在CustomerService接口的每个方法被调用时都在方法前执行一些逻辑，所以需要配置一个拦截器：

```java
package demo.long;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class CustomerServiceInterceptor {

    @Before("execution(* demo.long..*.*(..))")
    public void doBefore() {
        System.out.println("do some important things before..."); 
    }
}
```

把Bean加到Spring配置中

```xml
<aop:aspectj-autoproxy />

<bean id="customerService" class="demo.long.impl.CustomerServiceImpl" />
<bean id="customerServiceInterceptor" class="demo.long.CustomerServiceInterceptor" />
```

如果现在外部对象调用CustomerService的doSomething1()方法的时候，会发现只有doSomething1()方法执行前打印了“do some important things before...”，而doSomething1()内部调用doSomething2()时并没有打印上述内容；外部对象单独调用doSomething2()时会打印上述内容。

```java
public class CustomerServiceTest {

    @Autowired
    ICustomerService customerService;

    @Test
    public void testAOP() {
        customerService.doSomething1();
    }
}
```

## 原因分析

拦截器的实现原理就是动态代理，实现AOP机制。Spring 的代理实现有两种：一是基于 JDK Dynamic Proxy 技术而实现的；二是基于 CGLIB 技术而实现的。如果目标对象实现了接口，在默认情况下Spring会采用JDK的动态代理实现AOP，CustomerServerImpl正是这种情况。

JDK动态代理生成的CustomerServiceImpl的代理类大致如下：

```java
public class CustomerServiceProxy implements CustomerService {  
  
    private CustomerService customerService;  
  
    public void setCustomerService(CustomerService customerService) {  
        this.customerService = customerService;  
    }  
  
    public void doSomething1() {  
        doBefore();  
        customerService.doSomething1();  
    }  
  
    public void doSomething2() {  
        doBefore();  
        customerService.doSomething2();  
    }  
  
    private void doBefore() {  
        // 例如，可以在此处开启事务或记录日志
        System.out.println("do some important things before...");  
    }  
  
}  
```

客户端程序使用代理类对象去调用业务逻辑：

```java
public class TestProxy {  
      
    public static void main(String[] args) {  
        // 创建代理目标对象
        // 对于Spring来说，这一工作是由Spring容器完成的。  
        CustomerService serviceProxyTarget = new CustomerServiceImpl();  
  
        // 创建代理对象
        // 对于Spring来说，这一工作也是由Spring容器完成的。 
        CustomerServiceProxy serviceProxy = new CustomerServiceProxy();  
        serviceProxy.setCustomerService(serviceProxyTarget);  
        CustomerService serviceBean = (CustomerService) serviceProxy;  
  
        // 调用业务逻辑操作  
        serviceBean.doSomething1();  
    }  
}  
```

执行main方法，发现doSomething1()中调用doSomething2()方法的时候并未去执行CustomerServiceProxy类的doBefore()方法。其实doSomething2()等同于this.doSomething2()，在CustomerServiceImpl类中this关键字表示的是当前这个CustomerServiceImpl类的实例，所以程序会去执行CustomerServiceImpl对象中的doSomething2()方法，而不会去执行CustomerServiceProxy类对象中的 doSomething2()方法。

在使用Spring AOP的时候，我们从IOC容器中获取的Bean对象其实都是代理对象，而不是那些Bean对象本身，由于this关键字引用的并不是该Service Bean对象的代理对象，而是其本身，因此Spring AOP是不能拦截到这些被嵌套调用的方法的。

## 解决方案

1. 修改类，不要出现“自调用”的情况：这是Spring文档中推荐的“最佳”方案；
2. 若一定要使用“自调用”，那么this.doSomething2()替换为：((CustomerService) AopContext.currentProxy()).doSomething2()；此时需要修改spring的aop配置：

xml: 

```xml
<aop:aspectj-autoproxy expose-proxy="true" />
```

注解：

```java
@EnableAspectJAutoProxy(exposeProxy = true)
```



## 参考链接：

https://www.jianshu.com/p/6534945eb3b5
