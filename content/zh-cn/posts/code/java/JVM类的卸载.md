+++
title = "JVM类的卸载"
date = 2022-03-01T16:26:31+08:00
featured = false
comment = true
toc = true
reward = true
pinned = false
categories = [
  "编程思想"
]
tags = [
  "Java","JVM"
]
series = [
  ""
]
images = []

+++

类的卸载：由JVM自带的类加载器所加载的类，在JVM的生命周期中，始终不会被卸载。JVM本身会始终引用这些类加载器，而这些类加载器始终引用它们所加载的类的Class对象。所以说，这些Class对象始终是可触及的。 

由用户自定义的类加载器所加载的类是可以被卸载的。

<!--more--> 

当类被加载，连接和初始化后，它的生命周期就开始了。当代表类的Class对象不在被引用时，即不可触及时，Class对象就会结束生命周期，类在方法区内的数据也会被卸载，从而结束类的生命周期。由此可见，一个类何时结束生命周期，取决于代表它的Class对象何时结束生命周期。由Java虚拟机自带的类加载器所加载的类，在虚拟机的生命周期中，始终不会被卸载。包括根类加载器，扩展类加载器和系统类加载器，Java虚拟机本身会始终引用这些类加载器，而这些类加载器则会始终引用这些类加载器，而这些类加载器则会始终引用它们所加载的类的Class对象，因此这些Class对象始终是可触及的。

<img src="https://cdn.tkaid.com/img/1443749-20200420025519968-94458427.png" alt="img" style="zoom: 60%;" />

如果程序运行过程中，将上图左侧三个引用变量都置为null，此时Sample对象结束生命周期，MyClassLoader对象结束生命周期，代表Sample类的Class对象也结束生命周期，Sample类在方法区内的二进制数据被卸载。

当再次有需要时，会检查Sample类的Class对象是否存在，如果存在会直接使用，不再重新加载；如果不存在Sample类会被重新加载，在Java虚拟机的堆区会生成一个新的代表Sample类的Class实例(可以通过哈希码查看是否是同一个实例)。

  

通过虚拟机参数，可以查看类的加载与卸载过程

- `-verbose:class` 跟踪类的加载和卸载
- `-XX:+TraceClassLoading` 跟踪类的加载
- `-XX:+TraceClassUnloading` 跟踪类的卸载

 

（1）系统类加载器始终是不会被卸载的，示例如下：

添加虚拟机参数：`-verbose:class`

```java
System.out.println("第一次开始加载Sample类");
Class<?> clazz = ClassLoader.getSystemClassLoader().loadClass(Sample.class.getName());
Object obj = clazz.newInstance();
System.out.println(clazz.hashCode());    // 2018699554
obj = null;
clazz = null;
System.gc();
System.out.println("第二次开始加载Sample类");
clazz = ClassLoader.getSystemClassLoader().loadClass(Sample.class.getName());
System.out.println(clazz.hashCode());    // 2018699554
Thread.sleep(2000);
System.out.println("执行结束....");
```

控制台输出：

![img](https://cdn.tkaid.com/img/1443749-20200420025252004-1863456722.png)

此例也说明，由系统类加载器加载的类不会被卸载，并且只加载一次，Class对象也只有一个

 

（2）用户自定义类加载器可以卸载，示例如下

添加虚拟机参数：`-verbose:class`

```java
System.out.println("开始加载Sample类");
MyClassLoader myClassLoader = new MyClassLoader();
Class<?> clazz = myClassLoader.findClass(Sample.class.getName());
Object obj = clazz.newInstance();
// 当代表类的Class对象不在被引用时，Class对象就会结束生命周期，类在方法区内的数据也会被卸载
obj = null;
clazz = null;
myClassLoader = null;
System.gc();
Thread.sleep(2000);
System.out.println("执行结束....");
```

控制台输出：

![img](https://cdn.tkaid.com/img/1443749-20200420025349804-987473013.png)



参考

[JVM 类的卸载](https://www.cnblogs.com/caoxb/p/12735525.html)
