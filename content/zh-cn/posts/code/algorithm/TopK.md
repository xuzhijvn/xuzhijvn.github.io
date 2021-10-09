+++
title = "TopK"
description = ""
date = 2021-10-09T14:38:30+08:00
featured = false
comment = true
toc = true
reward = true
categories = [
  "编程思想"
]
tags = [
  "算法"
]
series = ["算法"]
images = []

+++



```java
public static int topK(int[] arr, int k) {
    if (arr == null || k > arr.length || arr.length == 0 || k <= 0) {
        return -1;
    }
    PriorityQueue<Integer> queue = new PriorityQueue<>(k, Comparator.reverseOrder());
    for (int i = 0; i < k; i++) {
        queue.add(arr[i]);
    }
    for (int i = k; i < arr.length; i++) {
        if (queue.peek() > arr[i]) {
            queue.poll();
            queue.add(arr[i]);
        }
    }
    return queue.peek();
}
```

