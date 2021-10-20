+++
title = "剑指 Offer 56 - I. 数组中数字出现的次数"
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
    public int[] singleNumbers(int[] nums) {
        if (nums == null) {
            return new int[0];
        }
        Set<Integer> set = new HashSet<>(2);
        for (int i = 0; i < nums.length; i++) {
            if (set.contains(nums[i])) {
                set.remove(nums[i]);
            } else {
                set.add(nums[i]);
            }
        }
        return set.stream().mapToInt(i -> i).toArray();
    }
}
```



[剑指 Offer 56 - I. 数组中数字出现的次数](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)

