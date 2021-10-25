---
title: "剑指 Offer 32 - II. 从上到下打印二叉树 II"
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

[comment]: <> "# 剑指 Offer 32 - II. 从上到下打印二叉树 II"



[剑指 Offer 32 - II. 从上到下打印二叉树 II](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/)

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        Queue<TreeNode> queue = new LinkedList<>();
        List<List<Integer>> res = new ArrayList<>();
        if(root != null) queue.add(root);
        while(!queue.isEmpty()) {
            List<Integer> tmp = new ArrayList<>();
            for(int i = queue.size(); i > 0; i--) {
                TreeNode node = queue.poll();
                tmp.add(node.val);
                if(node.left != null) queue.add(node.left);
                if(node.right != null) queue.add(node.right);
            }
            res.add(tmp);
        }
        return res;
    }
}
```

 

这种方法，要用到两个队列，但是思路比较清晰，也记录一下

```java
class Solution {
    LinkedList<TreeNode> queue1 = new LinkedList<>();
    LinkedList<TreeNode> queue2 = new LinkedList<>();
    
    public List<List<Integer>> levelOrder(TreeNode root) {

        List<List<Integer>> res = new ArrayList<>();
        if (root != null) {
            queue1.add(root);
        }
        while (!queue1.isEmpty()) {
            List<Integer> list = new ArrayList<>();
            while (!queue1.isEmpty()) {
                TreeNode node = queue1.poll();
                list.add(node.val);
                if (node.left != null) {
                    queue2.add(node.left);
                }
                if (node.right != null) {
                    queue2.add(node.right);
                }
            }
            res.add(list);
            LinkedList<TreeNode> tmp = queue1;
            queue1 = queue2;
            queue2 = tmp;
        }
        return res;
    }
}
```

