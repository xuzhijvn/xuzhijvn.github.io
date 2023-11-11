---
title: "Java Agent"
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

[comment]: <> "# Java agent"

 Java Agent就是一个可以作为java代理的工具, 简单来说就是一个可供用于编写的java切面, 它的主要功能**就是为用户提供了在 JVM 将字节码文件读入内存之后，JVM 使用对应的字节流在 Java 堆中生成一个 Class 对象之前，用户可以对其字节码进行修改的能力，从而 JVM 也将会使用用户修改过之后的字节码进行新的Class 对象的创建(打破了一个类只能加载一次的规则)。**

 Java Agent的使用对于你自身的代码是无侵入性的。应用场景：`热更新`。

热更新我们也可以自定义类加载器实现，这种方式的热更新是jvm原生支持的方式, 但是缺点也很明显:

1. 不够灵活, 需要手动修改文件等操作

2. 重复创建类加载器, 并且卸载困难, 会增加系统负担

3. 使用起来具有代码侵入性, 需要对代码进行一定改造

通过 Java Agent完美的解决了我们自定义类加载器实现热更新的缺点。

### 1.1 JVM启动前静态Instrument

通过启动命令 `java -javaagent:agent1.jar -javaagent:agent2.jar -jar MyProgram.jar` 在目标程序`main`方法执行前，先执行agent中定义的 `premain` 方法

### 1.2 JVM启动后动态Instrument

`Java6` 以后提供了在目标程序`main`方法执行后，执行`agent`的`agentmain`方法的机制，通过这种机制，我们可以动态修改目标程序已经加载过的字节码。

在`Java6` 以后实现启动后加载的新实现是`Attach API `。`Attach API` 很简单，只有 2 个主要的类，即`VirtualMachine` 和 `VirtualMachineDescriptor`，都在`tool.jar` 的 `com.sun.tools.attach` 包里面。

`attach`实现动态注入的原理如下：

通过`VirtualMachine`类的`attach(pid)`方法，便可以`attach`到一个运行中的`java进程`上，之后便可以通过`loadAgent(agentJarPath)`来将`agent`的`jar包`注入到对应的进程，然后对应的进程会调用`agentmain`方法。

<img src="https://cdn.tkaid.com/img/1607781-20190817155003876-767522290.png" alt="attach实现动态注入的原理" style="zoom: 40%;" />

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

