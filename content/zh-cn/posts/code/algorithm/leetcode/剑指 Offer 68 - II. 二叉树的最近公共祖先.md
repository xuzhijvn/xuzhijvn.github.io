+++
title = "剑指 Offer 68 - II. 二叉树的最近公共祖先"
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

**方法二：存储父节点**

**思路**

我们可以用哈希表存储所有节点的父节点，然后我们就可以利用节点的父节点信息从 p 结点开始不断往上跳，并记录已经访问过的节点，再从 q 节点开始不断往上跳，如果碰到已经访问过的节点，那么这个节点就是我们要找的最近公共祖先。

**算法**

1. 从根节点开始遍历整棵二叉树，用哈希表记录每个节点的父节点指针。
2. 从 p 节点开始不断往它的祖先移动，并用数据结构记录已经访问过的祖先节点。
3. 同样，我们再从 q 节点开始不断往它的祖先移动，如果有祖先已经被访问过，即意味着这是 p 和 q 的深度最深的公共祖先，即 LCA 节点。

**复杂度分析**

时间复杂度：O(N)，其中 NN 是二叉树的节点数。二叉树的所有节点有且只会被访问一次，从 p 和 q 节点往上跳经过的祖先节点个数不会超过 N，因此总的时间复杂度为 O(N)。

空间复杂度：O(N)，其中 NN 是二叉树的节点数。递归调用的栈深度取决于二叉树的高度，二叉树最坏情况下为一条链，此时高度为 N，因此空间复杂度为 O(N)，哈希表存储每个节点的父节点也需要 O(N) 的空间复杂度，因此最后总的空间复杂度为 O(N)。



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

   Map<Integer, TreeNode> parents = new HashMap<>();
   Set<Integer> set = new HashSet<>();

    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {

        if (root == null || p == null || q == null) {
            return null;
        }
        if (root == p || root == q) {
            return root;
        }
        dfs(root);
        TreeNode res = null;
        TreeNode pp = p;
        TreeNode qp = q;
        set.add(pp.val);
        while (parents.get(pp.val) != null) {
            set.add(parents.get(pp.val).val);
            pp = parents.get(pp.val);
        }
        while (qp != null) {
            if (!set.contains(qp.val)) {
                set.add(qp.val);
                qp = parents.get(qp.val);
            } else {
                res = qp;
                break;
            }
        }
        return res;
    }

    public void dfs(TreeNode root){
        if (root == null){
            return;
        }
        if (root.left != null){
            parents.put(root.left.val, root);
            dfs(root.left);
        }
        if (root.right != null){
            parents.put(root.right.val, root);
            dfs(root.right);
        }
    }
}
```



[剑指 Offer 68 - II. 二叉树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

