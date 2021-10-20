+++
title = "剑指 Offer 68 - I. 二叉搜索树的最近公共祖先"
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

**方法二：一次遍历**
**思路与算法**

在方法一中，我们对从根节点开始，通过遍历找出到达节点 pp 和 qq 的路径，一共需要两次遍历。我们也可以考虑将这两个节点放在一起遍历。

整体的遍历过程与方法一中的类似：

我们从根节点开始遍历；

如果当前节点的值大于 pp 和 qq 的值，说明 pp 和 qq 应该在当前节点的左子树，因此将当前节点移动到它的左子节点；

如果当前节点的值小于 pp 和 qq 的值，说明 pp 和 qq 应该在当前节点的右子树，因此将当前节点移动到它的右子节点；

如果当前节点的值不满足上述两条要求，那么说明当前节点就是「分岔点」。此时，pp 和 qq 要么在当前节点的不同的子树中，要么其中一个就是当前节点。

可以发现，如果我们将这两个节点放在一起遍历，我们就省去了存储路径需要的空间。



```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {

        if (root == null || p == null || q == null) {
            return null;
        }
        if (root.val > p.val && root.val > q.val) {
            return lowestCommonAncestor(root.left, p, q);
        }
        if (root.val < p.val && root.val < q.val) {
            return lowestCommonAncestor(root.right, p, q);
        }
        return root;
    }
}
```



**复杂度分析**

- 时间复杂度：O(n)*O*(*n*)，其中 n*n* 是给定的二叉搜索树中的节点个数。分析思路与方法一相同。
- 空间复杂度：O(1)*O*(1)。



[剑指 Offer 68 - I. 二叉搜索树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

