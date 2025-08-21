---
title: 路径总和 II
date: 2022-06-02 14:52:06
tags:
  - leetcode
  - 算法
  - 树
---

[路径总和 II](https://leetcode.cn/problems/path-sum-ii/)

给你二叉树的根节点 root 和一个整数目标和 targetSum ，找出所有 从根节点到叶子节点 路径总和等于给定目标和的路径。

叶子节点 是指没有子节点的节点。

<!--more-->

![pathsumii1](pathsumii1.jpg)

```
输入：root = [5,4,8,11,null,13,4,7,2,null,null,5,1], targetSum = 22
输出：[[5,4,11,2],[5,8,4,5]]
```

思路：求root节点开始到叶子节点和为target的路径，等于当前节点加上子节点到叶子节点和为：target - root.val的路径。采用先序遍历，在遍历过程维护当前路径与子路径并及时回退

代码：

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    // 全局结果
    List<List<Integer>> result = new ArrayList();
    public List<List<Integer>> pathSum(TreeNode root, int targetSum) {
        if(root == null) {
            return result;
        }
        // 临时路径
        List<Integer> tmp = new ArrayList();
        findPath(root, targetSum, tmp);
        return result;
    }

    public void findPath(TreeNode node, int target, List<Integer> tmp){
        int val = node.val;
        // 添加当前节点路径
        tmp.add(val);
        // 计算子节点需要满足的target
        target -= val;
        // 叶子节点
        if(node.left == null && node.right == null) {
            if(target == 0) {  // found matched
                List<Integer> t = new ArrayList();
                t.addAll(tmp);
                result.add(t);
            }
        } else{
            // 找左子树
            if(node.left != null) {
                findPath(node.left, target, tmp);
            }
            if(node.right != null) {
                findPath(node.right, target, tmp);
            }
        }
				// 回退临时路径
        tmp.remove(tmp.size()-1);
    }
}
```

