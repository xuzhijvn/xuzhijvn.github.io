+++
title = "剑指 Offer 58 - I. 翻转单词顺序"
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





**方法一：双指针**

**算法解析：**

- 倒序遍历字符串 s ，记录单词左右索引边界 i , j ；
- 每确定一个单词的边界，则将其添加至单词列表 res ；
- 最终，将单词列表拼接为字符串，并返回即可。

复杂度分析：

- 时间复杂度 O(N) ： 其中 N为字符串 s 的长度，线性遍历字符串。
- 空间复杂度 O(N)： 新建的 list(Python) 或 StringBuilder(Java) 中的字符串总长度  ≤N ，占用 O(N)大小的额外空间。



```java
class Solution {
    public String reverseWords(String s) {
        if (s == null) {
            return null;
        }
        s = s.trim();
        StringBuilder sb = new StringBuilder();
        for (int pos1 = s.length() - 1, pos2 = s.length() - 1; pos1 >= 0; pos1--) {
            if (s.charAt(pos1) == ' ' || pos1 == 0) {
                if (pos1 != 0 && s.charAt(pos1 + 1) == ' ') {
                    continue;
                }
                sb.append(s.substring(pos1, pos2 + 1).trim()).append(" ");
                pos2 = pos1;
            }
        }
        return sb.toString().trim();
    }
}
```



[剑指 Offer 58 - I. 翻转单词顺序](https://leetcode-cn.com/problems/fan-zhuan-dan-ci-shun-xu-lcof/)

