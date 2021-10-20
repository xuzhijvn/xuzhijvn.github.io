+++
title = "剑指 Offer 10- II. 青蛙跳台阶问题"
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
    public int numWays(int n) {
        if(n < 2){
            return 1;
        }
        if(n == 2){
            return n;
        }
        int a = 1, b = 2, sum = 2;
        for(int i = 2; i < n; i++){
            sum = (a + b) % 1000000007;
            a = b;
            b = sum;
        }
        return sum;
    }
}
```



[剑指 Offer 10- II. 青蛙跳台阶问题](https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)

