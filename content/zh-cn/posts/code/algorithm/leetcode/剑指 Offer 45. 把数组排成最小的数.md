+++
title = "剑指 Offer 45. 把数组排成最小的数"
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
public static String minNumber(int[] nums) {

            //to IntStream
    return Arrays.stream(nums)
            //to Stream<Integer>
            .boxed()
            //Stream<String>
            .map(String::valueOf)
            //排序
            .sorted((o1, o2) -> (o1 + o2).compareTo(o2 + o1))
            //to String
            .collect(Collectors.joining(""));

}
```

[剑指 Offer 45. 把数组排成最小的数](https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/)

