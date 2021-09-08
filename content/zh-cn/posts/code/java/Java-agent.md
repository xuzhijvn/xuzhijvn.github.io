---
title: "Java agent"
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
"images/center.png"
]
---

[comment]: <> "# Java agent"

**开发者可以构建一个独立于应用程序的代理程序（Agent），用来监测和协助运行在 JVM 上的程序，甚至能够替换和修改某些类的定义。**有了这样的功能，开发者就可以实现更为灵活的运行时虚拟机监控和 Java 类操作了，这样的特性实际上提供了 一种虚拟机级别支持的 AOP 实现方式，使得开发者无需对 JDK 做任何升级和改动，就可以实现某些 AOP 的功能了。

### 1.1 JVM启动前静态Instrument

通过启动命令 `java -javaagent:agent1.jar -javaagent:agent2.jar -jar MyProgram.jar` 在目标程序`main`方法执行前，先执行agent中定义的 `premain` 方法

### 1.2 JVM启动后动态Instrument

`Java6` 以后提供了在目标程序`main`方法执行后，执行`agent`的`agentmain`方法的机制，通过这种机制，我们可以动态修改目标程序已经加载过的字节码。

在`Java6` 以后实现启动后加载的新实现是`Attach API `。`Attach API` 很简单，只有 2 个主要的类，即`VirtualMachine` 和 `VirtualMachineDescriptor`，都在`tool.jar` 的 `com.sun.tools.attach` 包里面。

`attach`实现动态注入的原理如下：

通过`VirtualMachine`类的`attach(pid)`方法，便可以`attach`到一个运行中的`java进程`上，之后便可以通过`loadAgent(agentJarPath)`来将`agent`的`jar包`注入到对应的进程，然后对应的进程会调用`agentmain`方法。

![attach实现动态注入的原理](https://picgo.6and.ltd/img/1607781-20190817155003876-767522290.png)

既然是两个进程之间通信那肯定的建立起连接，`VirtualMachine.attach`动作类似TCP创建连接的三次握手，目的就是搭建`attach`通信的连接。而后面执行的操作，例如`vm.loadAgent`，其实就是向这个`socket`写入数据流，接收方`target VM`会针对不同的传入数据来做不同的处理。

例如：找到当前`JVM`并加载`agent.jar`（即`attach JVM` 和 `runing JVM` 是同一个 `JVM`，真正的应用中更多的是不同的两个`JVM`，这里仅为了测试方便。）

```java
package com.rickiyang.learn.job;

import com.sun.tools.attach.*;

import java.io.IOException;
import java.util.List;

/**
 * @author rickiyang
 * @date 2019-08-16
 * @Desc
 */
public class TestAgentMain {

    public static void main(String[] args) throws IOException, AttachNotSupportedException, AgentLoadException, AgentInitializationException {
        //获取当前系统中所有 运行中的 虚拟机
        System.out.println("running JVM start ");
        List<VirtualMachineDescriptor> list = VirtualMachine.list();
        for (VirtualMachineDescriptor vmd : list) {
            //如果虚拟机的名称为 xxx 则 该虚拟机为目标虚拟机，获取该虚拟机的 pid
            //然后加载 agent.jar 发送给该虚拟机
            System.out.println(vmd.displayName());
            if (vmd.displayName().endsWith("com.rickiyang.learn.job.TestAgentMain")) {
                VirtualMachine virtualMachine = VirtualMachine.attach(vmd.id());
                virtualMachine.loadAgent("/Users/yangyue/Documents/java-agent.jar");
                virtualMachine.detach();
            }
        }
    }

}
```



## 参考

[javaagent 应用 | 七日打卡](https://juejin.cn/post/6917522279303741453)

[javaagent使用指南](https://www.cnblogs.com/rickiyang/p/11368932.html)

[基于Java Instrument的Agent实现](https://juejin.cn/post/6844903587156328455)

