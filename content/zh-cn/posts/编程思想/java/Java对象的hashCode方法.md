---
title: "Java对象的hashCode方法"
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

[comment]: <> (# Java对象的hashCode方法)

java中的Object类的hashCode方法是一个native方法，查看native源码过于困难，所以暂且认为 Object类的hashCode生成规则是：hash(对象的内存地址+一些其他信息)

java中String类的 hashCode方法 比较直观，源码如下：

```java
    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
```

生成规则：*s[0]\*31^(n-1) + s[1]\*31^(n-2) + ... + s[n-1]*

为什么是素数31？

素数：根据素数的特点，一个数与素数相乘，得到结果只能被1、这个数、素数本身整除。因此，按照 *s[0]\*31^(n-1) + s[1]\*31^(n-2) + ... + s[n-1]* 生成的hashCode越不容易发生碰撞。

31：哈希计算速度快。可用移位和减法来代替乘法。现代的VM可以自动完成这种优化，如31 * i = (i << 5) - i。

参考：

https://juejin.im/post/5ba75d165188255c6a043b96
