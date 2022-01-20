---
title: "OOM实战"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"Java","JVM"
]
series : [
"Manual"
]
images : [


]
---

[comment]: <> "# OOM实战"

《深入理解Java虚拟机》中将OOM划分为: `Java堆溢出`、`虚拟机栈和本地方法栈溢出`、`方法区和运行时常量池溢出`、`本机直接内存溢出`

## 1. Java堆溢出

```java
/**
 * JDK1.6/JDK1.8
 * 
 * Java堆内存溢出异常测试
 * 
 * VM Args: -Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
 * 
 * @author xuzhijun.online  
 * @date 2019年4月22日
 */
public class HeapOOM {
  
  static class OOMObject{
    
  }

  public static void main(String[] args) {
    List<OOMObject> list = new ArrayList<OOMObject>();
    while(true) {
      list.add(new OOMObject());
    }
  }

}
```

运行结果：

```shell
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid3404.hprof ...
Heap dump file created [22045981 bytes in 0.663 secs]
```

处理方法：

1. 首先通过内存映像分析工具（如Eclipse Memory Analyzer） 对Dump出来的堆转储快照进行分析。 分清楚到底是出现了内存泄漏（Memory Leak） 还是内存溢出（Memory Overflow） 。
2. 如果是内存泄漏， 可进一步通过工具查看泄漏对象到GC Roots的引用链， 找到泄漏对象是通过怎样的引用路径、 与哪些GC Roots相关联， 才导致垃圾收集器无法回收它们， 根据泄漏对象的类型信息以及它到GC Roots引用链的信息， 一般可以比较准确地定位到这些对象创建的位置， 进而找出产生内存泄漏的代码的具体位置。
3. 如果是内存溢出， 换句话说就是内存中的对象确实都是必须存活的， 那就应当检查Java虚拟机的堆参数（-Xmx与-Xms） 设置， 与机器的内存对比， 看看是否还有向上调整的空间。 再从代码上检查是否存在某些对象生命周期过长、 持有状态时间过长、 存储结构设计不合理等情况， 尽量减少程序运行期的内存消耗。

## 2. 虚拟机栈和本地方法栈溢出

由于HotSpot虚拟机中并不区分虚拟机栈和本地方法栈， 因此对于HotSpot来说， -Xoss参数（设置本地方法栈大小） 虽然存在， 但实际上是没有任何效果的， 栈容量只能由-Xss参数来设定。 关于虚拟机栈和本地方法栈， 在《Java虚拟机规范》 中描述了两种异常：

1） 如果线程请求的栈深度大于虚拟机所允许的最大深度， 将抛出StackOverflowError异常。

2） 如果虚拟机的栈内存允许动态扩展， 当扩展栈容量无法申请到足够的内存时， 将抛出OutOfMemoryError异常。

《Java虚拟机规范》 明确允许Java虚拟机实现自行选择是否支持栈的动态扩展， 而HotSpot虚拟机的选择是不支持扩展， 所以除非在创建线程申请内存时就因无法获得足够内存而出现OutOfMemoryError异常， 否则在线程运行时是不会因为扩展而导致内存溢出的， 只会因为栈容量无法容纳新的栈帧而导致StackOverflowError异常。

无法容纳新的栈帧有两种情况：1. `栈帧太深` 2. `单个栈帧太大`。分别测试如下：

### 2.1 栈帧太深

```java
/**
 * JDK1.8/JDK1.6
 * 
 * 虚拟机机栈和本地方法栈OOM测试(减小栈内存，模拟StackOverflowError)
 * 
 * VM Args: -Xss128k
 * 
 * @author 106.55.152.92:30989
 * @date 2019年4月23日
 */
public class JavaVMStackSOF {
  
  private int stackLength = 1;
  
  public void stackLeak() {
    stackLength++;
    stackLeak();
  }

  public static void main(String[] args) throws Throwable {
    JavaVMStackSOF oom = new JavaVMStackSOF();
    try {
      oom.stackLeak();
    } catch (Throwable e) {
      System.out.println("stack length: "+oom.stackLength);
      throw e;
    }
  }

}
```

运行结果：

```shell
stack length:2402
Exception in thread "main" java.lang.StackOverflowError
at org.fenixsoft.oom. JavaVMStackSOF.leak(JavaVMStackSOF.java:20)
at org.fenixsoft.oom. JavaVMStackSOF.leak(JavaVMStackSOF.java:21)
at org.fenixsoft.oom. JavaVMStackSOF.leak(JavaVMStackSOF.java:21)
……后续异常堆栈信息省略
```

 

### 2.2 单个栈帧太大

```java
public class JavaVMStackSOF {
    private static int stackLength = 0;

    public static void test() {
        long unused1, unused2, unused3, unused4, unused5,
                unused6, unused7, unused8, unused9, unused10,
                unused11, unused12, unused13, unused14, unused15,
                unused16, unused17, unused18, unused19, unused20,
                unused21, unused22, unused23, unused24, unused25,
                unused26, unused27, unused28, unused29, unused30,
                unused31, unused32, unused33, unused34, unused35,
                unused36, unused37, unused38, unused39, unused40,
                unused41, unused42, unused43, unused44, unused45,
                unused46, unused47, unused48, unused49, unused50,
                unused51, unused52, unused53, unused54, unused55,
                unused56, unused57, unused58, unused59, unused60,
                unused61, unused62, unused63, unused64, unused65,
                unused66, unused67, unused68, unused69, unused70,
                unused71, unused72, unused73, unused74, unused75,
                unused76, unused77, unused78, unused79, unused80,
                unused81, unused82, unused83, unused84, unused85,
                unused86, unused87, unused88, unused89, unused90,
                unused91, unused92, unused93, unused94, unused95,
                unused96, unused97, unused98, unused99, unused100;
        stackLength++;
        test();
        unused1 = unused2 = unused3 = unused4 = unused5 = 
        unused6 = unused7 = unused8 = unused9 = unused10 =
        unused11 = unused12 = unused13 = unused14 = unused15 = 
        unused16 = unused17 = unused18 = unused19 = unused20 =
        unused21 = unused22 = unused23 = unused24 = unused25 = 
        unused26 = unused27 = unused28 = unused29 = unused30 =
        unused31 = unused32 = unused33 = unused34 = unused35 =
        unused36 = unused37 = unused38 = unused39 = unused40 =
        unused41 = unused42 = unused43 = unused44 = unused45 = 
        unused46 = unused47 = unused48 = unused49 = unused50 =
        unused51 = unused52 = unused53 = unused54 = unused55 =
        unused56 = unused57 = unused58 = unused59 = unused60 =
        unused61 = unused62 = unused63 = unused64 = unused65 =
        unused66 = unused67 = unused68 = unused69 = unused70 =
        unused71 = unused72 = unused73 = unused74 = unused75 =
        unused76 = unused77 = unused78 = unused79 = unused80 =
        unused81 = unused82 = unused83 = unused84 = unused85 =
        unused86 = unused87 = unused88 = unused89 = unused90 =
        unused91 = unused92 = unused93 = unused94 = unused95 =
        unused96 = unused97 = unused98 = unused99 = unused100 = 0;
    }

    public static void main(String[] args) {
        try {
            test();
        } catch (Error e) {
            System.out.println("stack length:" + stackLength);
            throw e;
        }
    }
}
```

运行结果:

```shell
stack length:5675
Exception in thread "main" java.lang.StackOverflowError
at org.fenixsoft.oom. JavaVMStackSOF.leak(JavaVMStackSOF.java:27)
at org.fenixsoft.oom. JavaVMStackSOF.leak(JavaVMStackSOF.java:28)
at org.fenixsoft.oom. JavaVMStackSOF.leak(JavaVMStackSOF.java:28)
……后续异常堆栈信息省略
```

实验结果表明： 无论是由于栈帧太大还是虚拟机栈容量太小， 当新的栈帧内存无法分配的时候，HotSpot虚拟机抛出的都是StackOverflowError异常。 可是如果在允许动态扩展栈容量大小的虚拟机上， 相同代码则会导致不一样的情况。 譬如远古时代的Classic虚拟机， 这款虚拟机可以支持动态扩展栈内存的容量， 在Windows上的JDK 1.0.2运行上述代码（2.2 单个栈帧太大的代码）的话（如果这时候要调整栈容量就应该改用-oss参数了） ， 得到的结果是：

```shell
stack length:3716
java.lang.OutOfMemoryError
at org.fenixsoft.oom. JavaVMStackSOF.leak(JavaVMStackSOF.java:27)
at org.fenixsoft.oom. JavaVMStackSOF.leak(JavaVMStackSOF.java:28)
at org.fenixsoft.oom. JavaVMStackSOF.leak(JavaVMStackSOF.java:28)
……后续异常堆栈信息省略
```

### 2.3 不断的创建线程

如果测试时不限于单线程， 通过不断建立线程的方式， 在HotSpot上也是可以产生内存溢出异常的。 但是这样产生的内存溢出异常和栈空间是否足够并不存在任何直接的关系， 主要取决于操作系统本身的内存使用状态。 甚至可以说， 在这种情况下， 给每个线程的栈分配的内存越大， 反而越容易产生内存溢出异常。原因其实不难理解， 操作系统分配给每个进程的内存是有限制的， 譬如32位Windows的单个进程最大内存限制为2GB。 HotSpot虚拟机提供了参数可以控制Java堆和方法区这两部分的内存的最大值，那剩余的内存即为2GB（操作系统限制） 减去最大堆容量， 再减去最大方法区容量， 由于程序计数器消耗内存很小， 可以忽略掉， 如果把直接内存和虚拟机进程本身耗费的内存也去掉的话， 剩下的内存就由虚拟机栈和本地方法栈来分配了。 因此为每个线程分配到的栈内存越大， 可以建立的线程数量自然就越少， 建立线程时就越容易把剩下的内存耗尽。

```java
/**
 * JDK1.8/JDK1.6(window操作系统假死，未能观测到异常)
 * 
 * 创建线程导致内存溢出异常
 * 
 * VM Args: -Xss2M
 * 
 * @author 106.55.152.92:30989
 * @date 2019年4月23日
 */
public class JavaVMStackOOM {
  
  private void dontStop() {
    while(true) {
    }
  }

  public void stackLeakByThread() {
    while(true) {
      Thread thread = new Thread(new Runnable() {
        @Override
        public void run() {
          dontStop();
        }
      });
      thread.start();
    }
  }
  
  public static void main(String[] args) {
    JavaVMStackOOM oom = new JavaVMStackOOM();
    oom.stackLeakByThread();
  }

}
```

 

> 重点提示一下， 如果读者要尝试运行上面这段代码， 记得要先保存当前的工作， 由于在Windows平台的虚拟机中， Java的线程是映射到操作系统的内核线程上， 无限制地创建线程会对操作系统带来很大压力， 上述代码执行时有很高的风险， 可能会由于创建线程数量过多而导致操作系统假死。



运行结果:

```shell
Exception in thread "main" java.lang.OutOfMemoryError: unable to create native thread
```

处理方法：

1. 如果使用HotSpot虚拟机默认参数， 栈深度在大多数情况下到达1000~2000是完全没有问题， 对于正常的方法调用 ， 这个深度应该完全够用了， 出现StackOverflowError异常时检查业务逻辑是否合理。
2. 如果是建立过多线程导致的内存溢出， 在不能减少线程数量或者更换64位虚拟机的情况下， 就只能通过减少最大堆和减少栈容量来换取更多的线程。 

## 3. 方法区和运行时常量池溢出

由于运行时常量池是方法区的一部分， 所以这两个区域的溢出测试可以放到一起进行。

### 3.1 运行时常量池溢出

```
String::intern()是一个本地方法， 它的作用是如果字符串常量池中已经包含一个等于此String对象的字符串， 则返回代表池中这个字符串的String对象的引用； 否则， 会将此String对象包含的字符串添加到常量池中， 并且返回此String对象的引用。
```

在JDK 6或更早之前的HotSpot虚拟机中， 常量池都是分配在永久代中， 我们可以通过-XX： PermSize和-XX： MaxPermSize限制永久代的大小， 即可间接限制其中常量池的容量。

```java
/**
 * JDK1.6
 * 
 * 运行时常量池导致的内存溢出异常(JDK1.8之后去除了方法区, 常量池移入堆内存，因此本实验只能在JDK1.6上测试)
 * 
 * VM Args: -XX:PermSize=10M -XX:MaxPermSize=10M
 * 
 * @author 106.55.152.92:30989
 * @date 2019年4月23日
 */
public class RuntimeConstantPoolOOM {

  public static void main(String[] args) {
    //使用List保持着常量池的引用，避免Full GC回收常量池行为
    List<String> list = new ArrayList<String>();
    //10M的PermSize在integer范围内足够产生OOM了
    int i = 0;
    while(true) {
      list.add(String.valueOf(i++).intern());
    }
  }
}
```

运行结果：

```shell
Exception in thread "main" java.lang.OutOfMemoryError: PermGen space
at java.lang.String.intern(Native Method)
at org.fenixsoft.oom.RuntimeConstantPoolOOM.main(RuntimeConstantPoolOOM.java: 18)
```

因为自JDK 7起， 原本存放在永久代的字符串常量池被移至Java堆之中， 所以在JDK 7及以上版本， 限制方法区的容量对该测试用例来说是毫无意义的。 这时候使用-Xmx参数限制最大堆到6MB就能够看到以下两种运行结果之一， 具体取决于哪里的对象分配时产生了溢出：

```java
// OOM异常一：
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
at java.base/java.lang.Integer.toString(Integer.java:440)
at java.base/java.lang.String.valueOf(String.java:3058)
at RuntimeConstantPoolOOM.main(RuntimeConstantPoolOOM.java:12)
// OOM异常二：
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
at java.base/java.util.HashMap.resize(HashMap.java:699)
at java.base/java.util.HashMap.putVal(HashMap.java:658)
at java.base/java.util.HashMap.put(HashMap.java:607)
at java.base/java.util.HashSet.add(HashSet.java:220)
at RuntimeConstantPoolOOM.main(RuntimeConstantPoolOOM.java from InputFile-Object:14)
```

关于这个字符串常量池的实现在哪里出现问题， 还可以引申出一些更有意思的影响:

```java
/**
 * 
 * JDK1.8(true,false)/JDK1.6(false,false)
 * 
 * JDK1.8移除方法区，常量池加入到堆内存，所以new对象的地址和intern()返回的常量池中的地址是相同的(详情见《JVM虚拟机P57》)
 *
 * String.intern()返回引用测试
 *
 * @author 106.55.152.92:30989
 * @author 2019年4月23日
 *
 */

public class RuntimeConstantPoolOOM2 {

  public static void main(String[] args) {
    String str1 = new StringBuilder("计算机").append("软件").toString();
    System.out.println(str1.intern() == str1);
    
    String str2 = new StringBuilder("ja").append("va").toString();
    System.out.println(str2.intern() == str2);
  }

}
```

JDK 7 及其以上版本会得到(true,false)，而JDK6会得到(false,false)结果。

> 对str2比较返回false， 这是因为“java”这个字符串在执行String-Builder.toString()之前就已经出现过了，它是加载sun.misc.Vesion的时候加入常量池的， 字符串常量池中已经有它的引用， 不符合intern()方法要求“首次遇到”的原则， “计算机软件”这个字符串则是首次出现的，因此结果返回true。
>

### 3.2 方法区溢出

我们再来看看方法区的其他部分的内容， 方法区的主要职责是用于存放类型的相关信息， 如类名、 访问修饰符、 常量池、 字段描述、 方法描述等。 对于这部分区域的测试， 基本的思路是运行时产生大量的类去填满方法区， 直到溢出为止。下面代码中笔者借助了CGLib直接操作字节码运行时生成了大量的动态类。

```java
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * 
 * JDK1.6
 * 
 * 借助CGLib使方法区出现内存溢出异常 
 * 
 *
 * VM  Args: -XX:PermSize=10M -XX:MaxPermSize=10M (JDK1.8之后去除了方法区, 类数据放入MetaSpace直接内存区域，该参数无效)
 *
 * @author 106.55.152.92:30989
 * @author 2019年4月23日
 *
 */

public class JavaMethodAreaOOM {
  
  static class OOMObject {
    
  }

  public static void main(String[] args) {
    while(true) {
      Enhancer enhancer = new Enhancer();
      enhancer.setSuperclass(OOMObject.class);
      enhancer.setUseCache(false);
      enhancer.setCallback(new MethodInterceptor() {
        @Override
        public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
          return proxy.invokeSuper(obj, args);
        }
      });
      enhancer.create();
    }
  }
}
```

在JDK 7以及以前版本中的运行结果：

```shell
Caused by: java.lang.OutOfMemoryError: PermGen space
at java.lang.ClassLoader.defineClass1(Native Method)
at java.lang.ClassLoader.defineClassCond(ClassLoader.java:632)
at java.lang.ClassLoader.defineClass(ClassLoader.java:616)
... 8 more
```

在JDK 8以后， 永久代便完全退出了历史舞台， 元空间作为其替代者登场。 在默认设置下， 前面列举的那些正常的动态创建新类型的测试用例已经很难再迫使虚拟机产生方法区的溢出异常了。 不过为了让使用者有预防实际应用里出现类似于上述代码那样的破坏性的操作， HotSpot还是提供了一些参数作为元空间的防御措施， 主要包括 `-XX：MaxMetaspaceSize` ` -XX：MetaspaceSize` `-XX：MinMetaspaceFreeRatio`

```java
import javassist.CtClass;
/**
 * Create by xuzhijun.online on 2019/5/7.
 */
public class MetaspaceOOM {

    /**
     * JVM参数:-XX:MetaspaceSize=10m -XX:MaxMetaspaceSize=10m
     */
    static javassist.ClassPool cp = javassist.ClassPool.getDefault();

    public static void main(String[] args) throws Exception{
        for (int i = 0; ; i++) {
            CtClass c = cp.makeClass("com.xzj.TreeNode" + i);
            if (c.isFrozen()){
                c.defrost();
            }
           Class cc =  c.toClass();
            System.out.println(cc +" " +i);
        }
    }
}
```

测试结果：

`D:\app\jdk1.8.0_251\bin\java.exe` 报Compressed class space OOM

```shell
class com.xzj.TreeNode3078 3078
Exception in thread "main" javassist.CannotCompileException: by java.lang.OutOfMemoryError: Compressed class space
  at javassist.ClassPool.toClass(ClassPool.java:1085)
  at javassist.ClassPool.toClass(ClassPool.java:1028)
  at javassist.ClassPool.toClass(ClassPool.java:986)
  at javassist.CtClass.toClass(CtClass.java:1079)
  at com.xzj.MetaspaceOOM.main(MetaspaceOOM.java:28)
Caused by: java.lang.OutOfMemoryError: Compressed class space
  at java.lang.ClassLoader.defineClass1(Native Method)
  at java.lang.ClassLoader.defineClass(ClassLoader.java:756)
  at java.lang.ClassLoader.defineClass(ClassLoader.java:635)
  at sun.reflect.GeneratedMethodAccessor1.invoke(Unknown Source)
  at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
  at java.lang.reflect.Method.invoke(Method.java:498)
  at javassist.ClassPool.toClass2(ClassPool.java:1098)
  at javassist.ClassPool.toClass(ClassPool.java:1079)
  ... 4 more
```

测试结果：

`D:\app\jdk1.8.0_71\bin\java.exe` 和 `D:\app\jdk1.8.0_20\bin\java.exe` 都是报Metaspace OOM

```shell
class com.xzj.TreeNode5681 5681
Exception in thread "main" javassist.CannotCompileException: by java.lang.OutOfMemoryError: Metaspace
  at javassist.ClassPool.toClass(ClassPool.java:1085)
  at javassist.ClassPool.toClass(ClassPool.java:1028)
  at javassist.ClassPool.toClass(ClassPool.java:986)
  at javassist.CtClass.toClass(CtClass.java:1079)
  at com.xzj.MetaspaceOOM.main(MetaspaceOOM.java:20)
Caused by: java.lang.OutOfMemoryError: Metaspace
  at java.lang.ClassLoader.defineClass1(Native Method)
  at java.lang.ClassLoader.defineClass(ClassLoader.java:760)
  at java.lang.ClassLoader.defineClass(ClassLoader.java:642)
  at sun.reflect.GeneratedMethodAccessor1.invoke(Unknown Source)
  at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
  at java.lang.reflect.Method.invoke(Method.java:497)
  at javassist.ClassPool.toClass2(ClassPool.java:1098)
  at javassist.ClassPool.toClass(ClassPool.java:1079)
  ... 4 more
```

 

## 4. 本机直接内存溢出

直接内存（Direct Memory） 的容量大小可通过-XX： MaxDirectMemorySize参数来指定， 如果不去指定， 则默认与Java堆最大值（由-Xmx指定） 一致， 下面代码越过了DirectByteBuffer类直接通过反射获取Unsafe实例进行内存分配。

```java
/**
 * VM Args： -Xmx20M -XX:MaxDirectMemorySize=10M
 *
 * @author zzm
 */
public class DirectMemoryOOM {
    private static final int _1MB = 1024 * 1024;

    public static void main(String[] args) throws Exception {
        Field unsafeField = Unsafe.class.getDeclaredFields()[0];
        unsafeField.setAccessible(true);
        Unsafe unsafe = (Unsafe) unsafeField.get(null);
        while (true) {
            unsafe.allocateMemory(_1MB);
        }
    }
}
```

运行结果：

```shell
Exception in thread "main" java.lang.OutOfMemoryError
at sun.misc.Unsafe.allocateMemory(Native Method)
at org.fenixsoft.oom.DMOOM.main(DMOOM.java:20)
```

处理方法： 一个明显的特征是在Heap Dump文件中不会看见有什么明显的异常情况， 如果读者发现内存溢出之后产生的Dump文件很小， 而程序中又直接或间接使用了DirectMemory（典型的间接使用就是NIO） ， 那就可以考虑重点检查一下直接内存方面的原因了。

