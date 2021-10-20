+++
title = "剑指 Offer 22. 链表中倒数第k个节点"
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
    public ListNode getKthFromEnd(ListNode head, int k) {
        if (head == null || k <= 0) {
            return null;
        }
        ListNode pos1 = head, pos2 = head;
        for (int i = 0; i < k - 1; i++) {
            if (pos1.next == null) {
                return null;
            }
            pos1 = pos1.next;
        }
        while (pos1.next != null) {
            pos1 = pos1.next;
            pos2 = pos2.next;
        }
        return pos2;
    }
}
```



[剑指 Offer 22. 链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)

