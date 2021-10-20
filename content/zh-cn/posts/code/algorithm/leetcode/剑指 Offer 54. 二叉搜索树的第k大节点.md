+++
title = "剑指 Offer 54. 二叉搜索树的第k大节点"
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



方法1

```java
class Solution {
    Integer res, k;
    public int kthLargest(TreeNode root, int k) {
        this.k = k;
        dfs(root);
        return res;
    }

    private void dfs(TreeNode root) {
        if (root == null || k == 0) {
            return;
        }
        dfs(root.right);

        if (--k == 0) {
            res = root.val;
            return;
        }

        dfs(root.left);
    }
}
```

方法2

```java
public int kthLargest(TreeNode root, int k) {
    PriorityQueue<Integer> queue = new PriorityQueue<>(k);
    traverse(root, queue, k);
    return queue.peek();
}

private void traverse(TreeNode root, PriorityQueue<Integer> queue, int k) {
    if (root == null) {
        return;
    }
    if (queue.size() < k) {
        queue.offer(root.val);
    } else if (queue.peek() < root.val) {
        queue.poll();
        queue.offer(root.val);
    }
    traverse(root.left, queue, k);
    traverse(root.right, queue, k);
}
```

[剑指 Offer 54. 二叉搜索树的第k大节点](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/)

