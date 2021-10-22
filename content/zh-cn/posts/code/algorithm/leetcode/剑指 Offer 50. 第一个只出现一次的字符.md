+++
title = "剑指 Offer 50. 第一个只出现一次的字符"
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

**方法二：有序哈希表**

在哈希表的基础上，有序哈希表中的键值对是 **按照插入顺序排序** 的。基于此，可通过遍历有序哈希表，实现搜索首个 “数量为 1的字符”。

哈希表是 **去重** 的，即哈希表中键值对数量 ≤ 字符串 s 的长度。因此，相比于方法一，方法二减少了第二轮遍历的循环次数。当字符串很长（重复字符很多）时，方法二则效率更高。

**复杂度分析：**

时间和空间复杂度均与 “方法一” 相同，而具体分析：方法一 需遍历 s 两轮；方法二 遍历 s 一轮，遍历dic一轮（dic 的长度不大于 26 ）。

Java 使用 `LinkedHashMap` 实现有序哈希表。



```java
public char firstUniqChar(String s) {
    LinkedHashMap<Character, Integer> map = new LinkedHashMap<>();
    for (int i = 0; i < s.length(); i++) {
        if (map.containsKey(s.charAt(i))) {
            map.put(s.charAt(i), map.get(s.charAt(i)) + 1);
        } else {
            map.put(s.charAt(i), 1);
        }
    }
    for (Map.Entry<Character, Integer> entry : map.entrySet()) {
        if (entry.getValue() == 1) {
            return entry.getKey();
        }
    }
    return ' ';
}
```



[剑指 Offer 50. 第一个只出现一次的字符](https://leetcode-cn.com/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/)

