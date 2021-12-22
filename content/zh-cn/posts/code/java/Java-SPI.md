+++
title = "Java SPI"
description = ""
date = 2021-12-22T16:49:52+08:00
featured = false
comment = true
toc = true
reward = true
pinned = false
categories = [
  "编程思想"
]
tags = [
  "Java"
]
series = [
  "Manual"
]
images = []

+++

本文不回答SPI是什么？而是结合源码深入剖析SPI该怎么使用。

<!--more-->

我们以JDBC为例分析，Java 定义好了 `java.sql.Driver`   实现的厂商有 `com.mysql.cj.jdbc.Driver`  、 `org.postgresql.Driver`

它们通过SPI机制被加载。

> 在JDBC4.0之前，我们开发有连接数据库的时候，通常会用`Class.forName("com.mysql.jdbc.Driver")`这句先加载数据库相关的驱动，然后再进行获取连接等的操作。而JDBC4.0之后不需要用`Class.forName("com.mysql.jdbc.Driver")`来加载驱动，直接获取连接就可以了，现在这种方式就是使用了Java的SPI扩展机制来实现。 

直接使用如下代码：

```java
String url = "jdbc:xxxx://xxxx:xxxx/xxxx";
Connection conn = DriverManager.getConnection(url,username,password);
.....
```

这里并没有涉及到spi的使用，接着看下面的解析。

`java.sql.DriverManager` 中有一个静态代码块如下：

```java
static {
    loadInitialDrivers();
    println("JDBC DriverManager initialized");
}
```

可以看到是加载实例化驱动的，接着看loadInitialDrivers方法：

```java
private static void loadInitialDrivers() {
  	//1. 从系统变量中获取有关驱动的定义
    String drivers;
    try {
        drivers = AccessController.doPrivileged(new PrivilegedAction<String>() {
            public String run() {
                return System.getProperty("jdbc.drivers");
            }
        });
    } catch (Exception ex) {
        drivers = null;
    }

    AccessController.doPrivileged(new PrivilegedAction<Void>() {
        public Void run() {
						//3. 使用SPI来获取驱动的实现。
            ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
            Iterator<Driver> driversIterator = loadedDrivers.iterator();
            try{
              	//4. 遍历使用SPI获取到的具体实现，实例化各个实现类。
                while(driversIterator.hasNext()) {
                    driversIterator.next();
                }
            } catch(Throwable t) {
            // Do nothing
            }
            return null;
        }
    });

    println("DriverManager.initialize: jdbc.drivers = " + drivers);

    //4. 根据第一步获取到的驱动列表来实例化具体实现类
    if (drivers == null || drivers.equals("")) {
        return;
    }
    String[] driversList = drivers.split(":");
    println("number of Drivers:" + driversList.length);
    for (String aDriver : driversList) {
        try {
            println("DriverManager.Initialize: loading " + aDriver);
            Class.forName(aDriver, true,
                    ClassLoader.getSystemClassLoader());
        } catch (Exception ex) {
            println("DriverManager.Initialize: load failed: " + ex);
        }
    }
}   
```

上面的代码主要步骤是：

- 从系统变量中获取有关驱动的定义。
- 使用SPI来获取驱动的实现。
- 遍历使用SPI获取到的具体实现，实例化各个实现类。
- 根据第一步获取到的驱动列表来实例化具体实现类。

我们主要关注2,3步，这两步是SPI的用法，首先看第二步，使用SPI来获取驱动的实现，对应的代码是：

```java
ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
```

这里没有去`META-INF/services`目录下查找配置文件，也没有加载具体实现类，做的事情就是封装了我们的接口类型和类加载器，并初始化了一个迭代器。

接着看第三步，遍历使用SPI获取到的具体实现，实例化各个实现类，对应的代码如下：

```java
//获取迭代器
Iterator<Driver> driversIterator = loadedDrivers.iterator();
//遍历所有的驱动实现
while(driversIterator.hasNext()) {
    driversIterator.next();
}
```

在遍历的时候，首先调用`driversIterator.hasNext()`方法，这里会搜索classpath下以及jar包中所有的`META-INF/services`目录下的`java.sql.Driver`文件，并找到文件中的实现类的名字。然后是调用`driversIterator.next();`方法，此时就会根据驱动名字具体实例化各个实现类了。

让我们看下`driversIterator.next();`方法

```java
public S next() {
    if (acc == null) {
        return nextService();
    } else {
        PrivilegedAction<S> action = new PrivilegedAction<S>() {
            public S run() { return nextService(); }
        };
        return AccessController.doPrivileged(action, acc);
    }
}
```

继续看 `nextService` 方法

```java
private S nextService() {
    if (!hasNextService())
        throw new NoSuchElementException();
    String cn = nextName;
    nextName = null;
    Class<?> c = null;
    try {
        c = Class.forName(cn, false, loader);
    } catch (ClassNotFoundException x) {
        fail(service,
             "Provider " + cn + " not found");
    }
    if (!service.isAssignableFrom(c)) {
        fail(service,
             "Provider " + cn  + " not a subtype");
    }
    try {
        S p = service.cast(c.newInstance());
        providers.put(cn, p);
        return p;
    } catch (Throwable x) {
        fail(service,
             "Provider " + cn + " could not be instantiated",
             x);
    }
    throw new Error();          // This cannot happen
}
```

直到这里都没看到哪里有实例化，只看到 `Class.forName(cn, false, loader);`

其实，就是在执行类加载的时候实例化的。这是 `java.sql.Driver` 强制要求的：

```java
/**
 *
 * <P>When a Driver class is loaded, it should create an instance of
 * itself and register it with the DriverManager. This means that a
 * user can load and register a driver by calling:
 * <p>
 * {@code Class.forName("foo.bah.Driver")}
 * <p>
 */
public interface Driver {
		//...
}
```

注释提到，在执行 `Class.forName("foo.bah.Driver")` 的时候必须 `实例化` 并且向DriverManager `注册`

让我们一起验证一下MySQL有没有遵循Java的强制要求：

```java
public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    public Driver() throws SQLException {
    }

    static {
        try {
            DriverManager.registerDriver(new Driver());
        } catch (SQLException var1) {
            throw new RuntimeException("Can't register driver!");
        }
    }
}
```

现在驱动就被找到并实例化了。

如果同时存在多个驱动（多个SPI实现），那DriverManager又该选择用哪个驱动呢？

源码面前无秘密㊙️，让我们看下存在多个驱动的情况下，DriverManager选择哪个驱动获得连接呢？

```java
//  Worker method called by the public getConnection() methods.
private static Connection getConnection(
    String url, java.util.Properties info, Class<?> caller) throws SQLException {
    
    //省略...

    for(DriverInfo aDriver : registeredDrivers) {
        // If the caller does not have permission to load the driver then
        // skip it.
        if(isDriverAllowed(aDriver.driver, callerCL)) {
            try {
                println("    trying " + aDriver.driver.getClass().getName());
                Connection con = aDriver.driver.connect(url, info);
                if (con != null) {
                    // Success!
                    println("getConnection returning " + aDriver.driver.getClass().getName());
                    return (con);
                }
            } catch (SQLException ex) {
                if (reason == null) {
                    reason = ex;
                }
            }

        } else {
            println("    skipping: " + aDriver.getClass().getName());
        }
    }
    
     //省略...
}
```

遍历驱动注册链表 `registeredDrivers` ，有一个驱动获得连接成功就返回。



## 参考

[高级开发必须理解的Java中SPI机制](https://www.jianshu.com/p/46b42f7f593c)

[Java常用机制 - SPI机制详解](https://pdai.tech/md/java/advanced/java-advanced-spi.html)

[对JDBC驱动注册--DriverManager.registerDriver和 Class.forName()的理解](https://blog.csdn.net/u013679744/article/details/56298283)

[JDBC简单示例代码](https://www.yiibai.com/jdbc/jdbc-sample-code.html)

