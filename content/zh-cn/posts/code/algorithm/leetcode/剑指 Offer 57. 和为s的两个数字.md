+++
title = "剑指 Offer 57. 和为s的两个数字"
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
    public int[] twoSum(int[] nums, int target) {
        int head = 0;
        int tail = nums.length - 1;
        while (head < tail) {
            if (nums[head] + nums[tail] == target) {
                return new int[]{nums[head], nums[tail]};
            }
            if (nums[head] + nums[tail] > target) {
                tail--;
                continue;
            }
            if (nums[head] + nums[tail] < target) {
                head++;
            }
        }
        return new int[0];
    }
}
```



[剑指 Offer 57. 和为s的两个数字](https://leetcode-cn.com/problems/he-wei-sde-liang-ge-shu-zi-lcof/)

