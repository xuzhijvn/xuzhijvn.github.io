+++
title = "剑指 Offer 25. 合并两个排序的链表"
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
public class Merge {

    public ListNode Merge(ListNode list1, ListNode list2) {
        if(list1 == null && list2 == null){
            return null;
        }
        if (list1 == null){
            return list2;
        }
        if (list2 == null){
            return list1;
        }
        ListNode res = null;
        if(list1.val < list2.val){
            res = list1;
            res.next = Merge(list1.next, list2);
        }else {
            res = list2;
            res.next = Merge(list1, list2.next);
        }
        return res;
    }
}
```



[剑指 Offer 25. 合并两个排序的链表](https://leetcode-cn.com/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/)

