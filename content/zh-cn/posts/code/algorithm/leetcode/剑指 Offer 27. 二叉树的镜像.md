---
title: "剑指 Offer 27. 二叉树的镜像"
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

[comment]: <> "# 剑指 Offer 27. 二叉树的镜像"



[剑指 Offer 27. 二叉树的镜像](https://leetcode-cn.com/problems/er-cha-shu-de-jing-xiang-lcof/submissions/)

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
    public TreeNode mirrorTree(TreeNode root) {
        if(root == null){
            return root;
        }
        //TreeNode posL = root.left;
        //TreeNode posR = root.right;
        //root.left = posR;
        //root.right = posL;
        TreeNode tmp = null;
        tmp = root.left;
        root.left = root.right;
        root.right = tmp;
        mirrorTree(root.left);
        mirrorTree(root.right);
        return root;
    }
}
```

 

