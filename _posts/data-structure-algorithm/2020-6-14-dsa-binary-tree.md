---
title: 二叉树（二叉搜索树）原理及其遍历
date: 2020-6-14 16:33:54
tags: 数据结构
---

## 存储

> 基于基础数据结构，数组或链表

- 数组，要求空间连续，适合满二叉树场景，不浪费空间
- 链表，不强要求空间连续，适合其他场景，也即常用场景

## 结构

- 二叉树：每个节点最多含有两个子树的树
- 二叉搜索树：每个节点左右子节点有序的树

## 遍历

```go
type node struct {
	val   int
	left  *node
	right *node
}

func (n node) show() {
	fmt.Printf("%d ", n.val)
}

func newNode(val int) *node {
	return &node{val, nil, nil}
}

type btree struct {
	root *node
}
```

> 二叉树

```go
func preOrder(n *node) {
	if n == nil {
		return
	}

	n.show() //前
	preOrder(n.left)
	preOrder(n.right)
}

func inOrder(n *node) {
	if n == nil {
		return
	}

	inOrder(n.left)
	n.show() //中
	inOrder(n.right)
}

func postOrder(n *node) {
	if n == nil {
		return
	}

	postOrder(n.left)
	postOrder(n.right)
	n.show() //后
}
```

> 深度优先：前、中、后，序遍历，暴力破解方法

```go
func levelOrder(root *node) {
	var q []*node
	var n *node

	q = append(q, root)

	for len(q) != 0  {
		n, q = q[0], q[1:]
		n.show()

		//分层，按序入队
		if n.left != nil {
			q = append(q, n.left)
		}

		if n.right != nil {
			q = append(q, n.right)
		}
	}
}
```

> 广度优先：层次遍历

## 参考

- https://zh.wikipedia.org/wiki/%E4%BA%8C%E5%8F%89%E6%A0%91
- https://github.com/xied5531-forked/Go/blob/master/data-structures/binary-tree/binary-tree.go
