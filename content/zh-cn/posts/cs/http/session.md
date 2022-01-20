+++
title = "session"
description = ""
date = 2021-09-09T20:36:27+08:00
featured = false
comment = true
toc = true
reward = true
categories = [
"计算机科学"
]
tags = [
"CS", "http"
]
series = [
"Manual"
]
images = [

]

+++

<!--more-->

**当浏览器关闭时，Session就被销毁了？**

我们知道Session是JSP的九大内置对象（也叫隐含对象）中的一个，它的作用是可以保存当前用户的状态信息，初学它的时候，认为Session的生命周期是从打开一个浏览器窗口发送请求到关闭浏览器窗口，但其实这种说法是不正确的！

**下面就具体的去解释：**

当用户第一次访问Web应用中支持Session的某个网页时，就会开始一个新的Session，那么接下来当用户浏览这个Web应用的不同网页时，始终处于一个Session中

**再详细些：**

当一个Session开始时，Servlet容器会创建一个HttpSession对象，那么在HttpSession对象中，可以存放用户状态的信息，Servlet容器为HttpSession对象分配一个唯一标识符即Sessionid，Servlet容器把Sessionid作为一种Cookie保存在客户端的浏览器中。用户每次发出Http请求时，Servlet容器会从HttpServletRequest对象中取出Sessionid,然后根据这个Sessionid找到相应的HttpSession对象，从而获取用户的状态信息。

以上就是Session的运行机制，但是还没有提到Session的生命周期，再往下了解！

**其实让Session结束生命周期，有以下两种办法：**

- 一个是Session.invalidate()方法，不过这个方法在实际的开发中，并不推荐，可能在强制注销用户的时候会使用；
- 一个是当前用户和服务器的交互时间超过默认时间后，Session会失效

我们知道Session是存在于服务器端的，当把浏览器关闭时，浏览器并没有向服务器发送，任何请求来关闭Session，自然Session也不会被销毁，但是可以做一点努力，在所有的客户端页面里使用js的window.onclose来监视浏览器的关闭动作，然后向服务器发送一个请求来关闭Session，但是这种做法在实际的开发中也是不推荐使用的，最正常的办法就是不去管它，让它等到默认的时间后，自动销毁

那么为什么当我们关闭浏览器后，就再也访问不到之前的session了呢？

其实之前的Session一直都在服务器端，而当我们关闭浏览器时，此时的Cookie是存在于浏览器的进程中的，当浏览器关闭时，Cookie也就不存在了。

**其实Cookie有两种:**

- 一种是存在于浏览器的进程中;
- 一种是存在于硬盘上；（主动指定了过期时间 `Expires` 或者有效期 `Max-Age` ）

而session的Cookie是存在于浏览器的进程中，那么这种Cookie我们称为会话Cookie，当我们重新打开浏览器窗口时，之前的Cookie中存放的Sessionid已经不存在了，此时服务器从HttpServletRequest对象中没有检查到sessionid，服务器会再发送一个新的存有Sessionid的Cookie到客户端的浏览器中，此时对应的是一个新的会话，而服务器上原先的session等到它的默认时间到之后，便会自动销毁。

**ps:**

当在同一个浏览器中同时打开多个标签，发送同一个请求或不同的请求，仍是同一个session;

当不在同一个窗口中打开相同的浏览器时，发送请求，仍是同一个session;

当使用不同的浏览器时，发送请求，即使发送相同的请求，是不同的session;

当把当前某个浏览器的窗口全关闭，再打开，发起相同的请求时，就是本文所阐述的，是不同的session,但是它和session的生命周期是没有关系的。

## 参考

[浏览器关闭后，Session会话结束了么？](https://blog.csdn.net/stanxl/article/details/47105051)

