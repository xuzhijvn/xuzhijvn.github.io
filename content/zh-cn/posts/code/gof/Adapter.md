+++
title = "Adapter"
description = ""
date = 2021-10-05T21:33:09+08:00
featured = false
draft = true
comment = true
toc = true
reward = true
categories = [
  ""
]
tags = [
  ""
]
series = []
images = []
+++

<!--more-->

```java
interface A {
    void say();
}

class AImpl implements A {
     @Override
     public void say() {
      System.out.println("我是中国人，我说中文");
     }
 }

interface B {
    void play();
}

class A2BAdapter implements B {

    private A a;

    public A2BAdapter(A a) {
        this.a = a;
    }

    @Override
    public void play() {
        a.say();
    }
}

public class Main {
    public static void main(String[] args) {

        A a = new AImpl();

        B b = new A2BAdapter(a);

        b.play();
    }
}
```



## 参考

[适配器模式](https://www.runoob.com/design-pattern/adapter-pattern.html)
