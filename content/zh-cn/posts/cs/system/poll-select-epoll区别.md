+++
title = "poll select epoll区别"
description = ""
date = 2021-09-08T09:38:02+08:00
featured = false
comment = true
toc = true
reward = true
categories = [
"计算机科学"
]
tags = [
"CS",
"System"
]
series = [
"Manual"
]
images = [
"images/center.png"
]

+++

<!--more-->

bio的缺点？

```java
public class TestSocket {
    public static void main(String[] args) throws Exception{
        ServerSocket server = new ServerSocket(8080);
        while (true){
            //阻塞等待连接
            Socket client = server.accept();
            //fork出一个线程调系统的recvfrom去读IO
            new Thread(() -> {
                try {
                    //读数据
                    InputStream in = client.getInputStream();
                    BufferedReader reader = new BufferedReader(new InputStreamReader(in));
                    while (true){
                        System.out.println(reader.readLine());
                    }
                }catch (IOException e){
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```

上面调recvfrom的时候发生在连接建立之后，并不代表一定有数据可读，所以如果有10000个连接，但是只有10个连接有数据可读的时候就比较悲剧，fork出了10000个线程，发生了10000次recvfrom系统调用，真正有效的却只有10次。

换言之，bio调用recvfrom的时机不对，不应该在连接建立的时候去调，最好等有有效数据时才调。

select

同步多路复用

入参数是文件fds（file descriptor set 描述符集合，即连接集合），调用一次select，由系统内核遍历10000个fd，返回哪些连接是可读状态，上面的例子在select方式下只发生1次select+10次recvfrom系统调用

poll

同步多路复用

和select类似，入参不一样，没有最大文件描述符数量的限制

epoll

事件驱动的poll

不用遍历10000个，而是基于操作系统中断机制，有数据可读的时候，网卡驱动发出cpu中断，然后再调recvfrom去同步读。



所以 多路复用（poll select epoll）非常适合长连接短数据的场景，例如即时通信。

## 参考

[一文搞懂select、poll和epoll区别](https://zhuanlan.zhihu.com/p/272891398)
