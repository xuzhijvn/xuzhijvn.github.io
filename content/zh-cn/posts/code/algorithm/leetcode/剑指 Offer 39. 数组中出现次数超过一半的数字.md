+++
title = "剑指 Offer 39. 数组中出现次数超过一半的数字"
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

1. **哈希表统计法：** 遍历数组 `nums` ，用 HashMap 统计各数字的数量，即可找出 众数 。此方法时间和空间复杂度均为 *O*(*N*) 。
2. **数组排序法：** 将数组 `nums` 排序，**数组中点的元素** 一定为众数。
3. **摩尔投票法：** 核心理念为 **票数正负抵消** 。此方法时间和空间复杂度分别为 O(N) 和 O(1) ，为本题的最佳解法。

```java
public static int majorityElement(int[] nums) {
    HashMap<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        if (map.containsKey(nums[i])) {
            map.put(nums[i], map.get(nums[i]) + 1);
        } else {
            map.put(nums[i], 1);
        }
    }
    int max = 0;
    Integer res = null;
    for (Map.Entry entry : map.entrySet()) {
        if ((Integer)entry.getValue() > max){
            max = (Integer)entry.getValue();
            res = (Integer) entry.getKey();
        }
    }
    return res;
}
```



```java
class Solution {
    public static int majorityElement(int[] nums) {
        Arrays.sort(nums);
        return nums[nums.length / 2];
    }
}
```



```java
class Solution {
    public int majorityElement(int[] nums) {
        int first = nums[0];
        int sum = 1;
        for (int i = 1; i < nums.length; i++) {
            if (sum == 0){
                first = nums[i];
                sum = 1;
            }else {
                sum = nums[i] == first ? sum + 1 : sum - 1;
            }
        }
        return first;
    }
}
```

[剑指 Offer 39. 数组中出现次数超过一半的数字](https://leetcode-cn.com/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/)

