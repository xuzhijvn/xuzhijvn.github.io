+++
title = "剑指 Offer 26. 树的子结构"
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
public class HasSubtree {
    public boolean HasSubtree(TreeNode root1, TreeNode root2) {
        if (root1 == null || root2 == null) {
            return false;
        }
        boolean res = false;
        if (root1.val == root2.val) {
            res = doesTree1haveTree2(root1, root2);
        }
        if (res){
            return true;
        }else {
            return HasSubtree(root1.left, root2) || HasSubtree(root1.right, root2);
        }
    }

    private boolean doesTree1haveTree2(TreeNode root1, TreeNode root2) {

        if (root1 == null && root2 == null) {
            return true;
        }
        if (root1 == null) {
            return false;
        }
        if (root2 == null) {
            return true;
        }

        return root1.val == root2.val && doesTree1haveTree2(root1.left, root2.left) && doesTree1haveTree2(root1.right, root2.right);
    }
}
```



[剑指 Offer 26. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)

