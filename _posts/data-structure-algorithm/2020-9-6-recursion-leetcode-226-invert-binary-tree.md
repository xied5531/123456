---
title: 递归：力扣226. 翻转二叉树
date: 2020-9-6 1:33:54
tags: 递归
---

# 递归：力扣226. 翻转二叉树

## 226. 翻转二叉树

翻转一棵二叉树。

示例：
```
输入：

     4
   /   \
  2     7
 / \   / \
1   3 6   9
输出：

     4
   /   \
  7     2
 / \   / \
9   6 3   1
```

## 递归

### 分解问题，分治

- 原问题：翻转以当前节点为根的二叉树
- 当前场景：交换左右子节点
- 子问题一：翻转以当前节点的左子节点为根的二叉树
- 子问题二：翻转以当前节点的右子节点为根的二叉树

### 函数定义

- 参数：当前根节点
- 返回：翻转后的树的根节点

`func invertTree(root *TreeNode) *TreeNode`

> 当前函数定义正好和题目函数定义一致

### 最简场景

当前节点为nil节点

返回：nil

### 整合结果

1. 交换左右子树根节点
2. 翻转左右子树

## 代码

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func invertTree(root *TreeNode) *TreeNode {
    if root == nil {
        return nil
    }

    tmp := root.Left
    root.Left = root.Right
    root.Right = tmp

    root.Left = invertTree(root.Left)
    root.Right = invertTree(root.Right)

    return root
}
```
