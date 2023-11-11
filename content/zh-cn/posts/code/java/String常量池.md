+++
title = "String常量池"
description = ""
date = 2021-09-05T11:54:01+08:00
featured = false
comment = true
toc = true
reward = true
categories = [
"编程思想"
]
tags =  [
"Java"
]
series =  [
"Manual"
]
images =  [


]

+++

<!--more-->

## 引言

在Java开发中不管是前后端交互的JSON串，还是数据库中的数据存储，我们常常需要使用到String类型的字符串。作为最常用也是最基础的引用数据类型，JVM为String提供了字符串常量池来提高性能，本篇文章我们一起从底层JVM中认识并学习字符串常量池的概念和设计原理。

## 字符串常量池由来

在日常开发过程中，字符串的创建是比较频繁的，而字符串的分配和其他对象的分配是类似的，需要耗费大量的时间和空间，从而影响程序的运行性能，所以作为最基础最常用的引用数据类型，Java设计者在JVM层面提供了字符串常量池。

### 实现前提

1. 实现这种设计的一个很重要的因素是：String类型是不可变的，实例化后，不可变，就不会存在多个同样的字符串实例化后有数据冲突；
2. 运行时，实例创建的全局字符串常量池中会有一张表，记录着常量池中每个唯一的字符串对象维护一个引用，当垃圾回收时，发现该字符串被引用时，就不会被回收。

### 实现原理

为了提高性能并减少内存的开销，JVM在实例化字符串常量时进行了一系列的优化操作：

1. 在JVM层面为字符串提供字符串常量池，可以理解为是一个缓存区；
2. 创建字符串常量时，JVM会检查字符串常量池中是否存在这个字符串；
3. 若字符串常量池中存在该字符串，则直接返回引用实例；若不存在，先实例化该字符串，并且，将该字符串放入字符串常量池中，以便于下次使用时，直接取用，达到缓存快速使用的效果。

```java
String str1 = "abc";
String str2 = "abc";
System.out.println("str1 == str2: " + (str1 == str2)); //结果：str1 == str2: true
```

## 字符串常量池位置变化

### 方法区

提到字符串常量池，还得先从方法区说起。方法区和Java堆一样（但是方法区是`非堆`），是各个线程共享的内存区域，是用于存储已经被**JVM加载的类信息、常量、静态变量、即时编译器编译后的代码**等数据。
很多人会把方法区称为`永久代`，其实本质上是不等价的，只不过HotSpot虚拟机设计团队是选择把GC分代收集扩展到了方法区，使用永久代来代替实现方法区。其实，在方法区中的垃圾收集行为还是比较少的，这个区域的内存回收目标主要是针对常量池的回收和对类型的卸载，但是这个区域的回收总是不尽如人意的，如果该区域回收不完全就会出现内存泄露。当然，对于`JDK1.8`时，HostSpot VM对JVM模型进行了改造，将`元数据放到本地内存`，将`常量池和静态变量放到了Java堆`里。

### 元空间

JDK 1.8, HotSpot JVM将永久代移除了，使用本地内存来存储类的元数据信息，即为`元空间（Metaspace）`

所以，字符串常量池的具体位置是在哪里？当然这个我们后面需要区分jdk的版本，jdk1.7之前，jdk1.7，以及jdk1.8，因为这些版本中，字符串常量池因为方法区的改变而做了一些变化。

### JDK1.7之前

在jdk1.7之前，常量池是存放在方法区中的。

<img src="https://cdn.tkaid.com/img/20201126173632432.png" alt="JDK1.7之前" style="zoom:50%;" />

### JDK1.7

在jdk1.7中，字符串常量池移到了堆中，运行时常量池还在方法区中。

<img src="https://cdn.tkaid.com/img/20201126173618236.png" alt="JDK1.7" style="zoom:50%;" />

### JDK1.8

jdk1.8删除了永久代，方法区这个概念还是保留的，但是方法区的实现变成了`元空间`，常量池沿用jdk1.7，还是放在了堆中。这样的效果就变成了：常量池和静态变量存储到了堆中，类的元数据及运行时常量池存储到元空间中。

<img src="https://cdn.tkaid.com/img/20201126172724596.png" alt="JDK1.8" style="zoom:50%;" />

为啥要把方法区从JVM内存（永久代）移到直接内存（元空间)？主要有两个原因：

1. 直接内存属于本地系统的IO操作，具有更高的一个IO操作性能，而JVM的堆内存这种，如果有IO操作，也是先复制到直接内存，然后再去进行本地IO操作。经过了一系列的中间流程，性能就会差一些。非直接内存操作：`本地IO操作——>直接内存操作——>非直接内存操作——>直接内存操作——>本地IO操作`，而直接内存操作：`本地IO操作——>直接内存操作——>本地IO操作`。
2. 永久代有一个无法调整更改的JVM固定大小上限，回收不完全时，会出现`OutOfMemoryError`问题；而直接内存（元空间）是受到本地机器内存的限制，不会有这种问题。

### 变化

1. 在JDK1.7前，运行时常量池+字符串常量池是存放在方法区中，HotSpot VM对方法区的实现称为永久代。
2. 在JDK1.7中，字符串常量池从方法区移到堆中，运行时常量池保留在方法区中。
3. 在JDK1.8中，HotSpot移除永久代，使用元空间代替，此时字符串常量池保留在堆中，运行时常量池保留在方法区中，只是实现不一样了，JVM内存变成了直接内存。

## 结合代码

### 代码示例

```java
String str1 = "123";
String str2 = "123";
String str3 = "123";
String str4 = new String("123");
String str5 = new String("123");
String str6 = new String("123");
```

**结果**：

```java
str1 == str2：true
str2 == str3：true
str3 == str4：false
str4 == str5：false
str5 == str6：false
```

### jvm存储示例

<img src="https://cdn.tkaid.com/img/20201130112956433.png" alt="在这里插入图片描述" style="zoom:50%;" />

### 创建对象流程

对于jvm底层，`String str = new String("123")`创建对象流程是什么？

1. 在常量池中查找是否存在"123"这个字符串；若有，则返回对应的引用实例；若无，则创建对应的实例对象；
2. 在堆中new一个String类型的"123"字符串对象；
3. 将对象地址复制给str，然后创建一个应用。

**注意**：
若常量池里没有"123"字符串，则创建了2个对象；若有该字符串，则创建了一个对象及对应的引用。

## Q&A

### String str ="ab" + "cd";对象个数？

分析：若字符串常量池无该字符串对象

1. 字符串常量池：（1个对象）"abcd";
2. 堆：无
3. 栈：（1个引用）str
   总共：1个对象+1个引用

### String str = new String("abc");对象个数？

分析：若字符串常量池无该字符串对象

1. 字符串常量池：（1个对象）"abc";
2. 堆：（1个对象）new String("abc")
3. 栈：（1个引用）str
   总共：2个对象+1个引用

### String str = new String("a" + "b");对象个数？

分析：若字符串常量池无该字符串对象

1. 字符串常量池：（3个对象）"a"，"b"，"ab";
2. 堆：（1个对象）new String("ab")
3. 栈：（1个引用）str
   总共：4个对象+1个引用

### String str = new String("ab") + "ab";对象个数？

分析：若字符串常量池无该字符串对象

1. 字符串常量池：（1个对象）"ab";
2. 堆：（1个对象）new String("ab")
3. 栈：（1个引用）str
   总共：2个对象+1个引用

### String str = new String("ab") + new String("ab");对象个数？

分析：若字符串常量池无该字符串对象

1. 字符串常量池：（1个对象）"ab";
2. 堆：（2个对象）new String("ab")，new String("ab")
3. 栈：（1个引用）str
   总共：3个对象+1个引用

### String str = new String("ab") + new String("cd");对象个数？

分析：若字符串常量池无该字符串对象

1. 字符串常量池：（2个对象）"ab"，"cd";
2. 堆：（2个对象）new String("ab")，new String("cd")
3. 栈：（1个引用）str
   总共：4个对象+1个引用

### String str3 = str1 + str2;对象个数？

```java
String str1 = "ab";
String str2 = "cd";
String str3 = str1 + str2;
```

分析：若字符串常量池无该字符串对象

1. 字符串常量池：（3个对象）"ab"，"cd"，"abcd";
2. 堆：无
3. 栈：（3个引用）str1，str2，str3
   总共：3个对象+3个引用

### 如何指向字符串池中特定的对象？

通过`intern()`方法。
**代码**

```java
String str1 = "123";
String str2 = new String("123");
String str3 = str2;
System.out.println("str1 == str2：" + (str1 == str2));
System.out.println("str1 == str3：" + (str1 == str3));

//通过java.lang.String.intern()方法指定字符串对象
String str4 = str1.intern();
System.out.println("str1 == str4：" + (str1 == str4));

String str5 = str2.intern();
System.out.println("str1 == str5：" + (str1 == str5));

String str6 = str3.intern();
System.out.println("str1 == str6：" + (str1 == str6));
```

**结果**：

```java
str1 == str2：false
str1 == str3：false
str1 == str4：true
str1 == str5：true
str1 == str6：true
```

## 参考

[JVM——字符串常量池详解](https://www.cnblogs.com/Andya/p/14067618.html)

