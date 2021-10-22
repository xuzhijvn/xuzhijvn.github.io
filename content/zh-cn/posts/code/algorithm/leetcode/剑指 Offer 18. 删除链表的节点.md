---
title: "剑指 Offer 18. 删除链表的节点"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"剑指offer"
]
series : [
"算法"
]
images : [
"images/center.png"
]
---

[comment]: <> "# 剑指 Offer 18. 删除链表的节点"



```java
class Solution {
    public ListNode deleteNode(ListNode head, int val) {

        ListNode myHead = new ListNode(-1);
        myHead.next = head;

        ListNode pos1 = head;
        ListNode pos2 = myHead;

        //定位
        while (pos1.val != val) {
            pos2 = pos1;
            pos1 = pos1.next;
        }
        //删除
        pos2.next = pos1.next;
        return myHead.next;
    }
}
```

[剑指 Offer 18. 删除链表的节点](https://leetcode-cn.com/problems/shan-chu-lian-biao-de-jie-dian-lcof/)
