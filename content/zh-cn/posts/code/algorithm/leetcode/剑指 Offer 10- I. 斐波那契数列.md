+++
title = "斐波那契数列"
description = ""
date = 2021-10-11T18:04:59+08:00
featured = false

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



```java
class Solution {
    public int fib(int n) {
        final int MOD = 1000000007;
        if (n == 0) {
            return 0;
        }
        if (n == 1) {
            return 1;
        }
        int first = 0, second = 1, res = 0;
        for (int i = 2; i <= n; i++) {
            res = (first + second) % MOD;
            first = second;
            second = res;
        }
        return res;
    }
}
```



[剑指 Offer 10- I. 斐波那契数列](https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof/)

