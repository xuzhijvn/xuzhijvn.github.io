---
title: "剑指 Offer 28. 对称的二叉树"
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

[comment]: <> (# 剑指 Offer 28. 对称的二叉树)



```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public boolean isSymmetric(TreeNode root) {
        if(root == null){return true;}
        return isSymmetric2(root.left, root.right);
    }
    private boolean isSymmetric2(TreeNode left, TreeNode right){
        if (left == null && right == null){return true;}
        if ((left == null && right != null) || (left !=null && right == null)){
            return false;
        }
        if (left.val == right.val){
            return isSymmetric2(left.left, right.right) && isSymmetric2(left.right, right.left);
        }
        return false;
    }
}
```

[剑指 Offer 28. 对称的二叉树](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/)
