---
title: "剑指 Offer 32 - III. 从上到下打印二叉树 III"
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
"Manual"
]
images : [
"images/center.png"
]
---

[comment]: <> (# 剑指 Offer 32 - III. 从上到下打印二叉树 III)



[剑指 Offer 32 - III. 从上到下打印二叉树 III](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/)

```java
package com.xzj;

import java.util.*;

/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public static List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if (root == null){return res;}
        Stack<TreeNode> stack1 = new Stack();
        Stack<TreeNode> stack2 = new Stack();
        stack1.push(root);
        while (stack1.isEmpty() == false || stack2.isEmpty() == false){
            List<Integer> sub = new ArrayList();
            while (stack1.isEmpty() == false){
                TreeNode node = stack1.pop();
                sub.add(node.val);
                if (node.left != null){
                    stack2.push(node.left);
                }
                if (node.right != null){
                    stack2.push(node.right);
                }
            }
            if (sub.isEmpty() == false){
                res.add(sub);
            }
            sub = new ArrayList();
            while (stack2.isEmpty() == false){
                TreeNode node = stack2.pop();
                sub.add(node.val);
                if (node.right != null){
                    stack1.push(node.right);
                }
                if (node.left != null){
                    stack1.push(node.left);
                }
            }
            if (sub.isEmpty() == false){
                res.add(sub);
            }
        }
        return res;
    }

    public static void main(String[] args) {
        TreeNode node1 = new TreeNode(3);
        TreeNode node2 = new TreeNode(9);
        TreeNode node3 = new TreeNode(20);
        TreeNode node4 = new TreeNode(15);
        TreeNode node5 = new TreeNode(7);

        node1.left = node2;
        node1.right = node3;
        node3.left = node4;
        node3.right = node5;
        System.out.println(levelOrder(node1));
        System.out.println((7 & 8));
        System.out.println(0 / 2);
    }
}
```

 

