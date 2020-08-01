---
title: 基础数据结构：数组和链表
date: 2020-7-19 09:46:18
tags: 数据结构
---

## 数据结构

> 数据结构的存储方式只有两种：数组（顺序存储）和链表（链式存储）

> 其他高级数据结构均是基于这两种基础数据结构封装而来


| 特点  | 数组  | 链表  |
|---|---|---|
| 存  | 连续、顺序  | 不连续、链式  |
| 增删  | 整段搬迁，保持顺序性，O(n)  | 知道前驱和后驱即可，O(1)  |
| 改  | 直接改值即可  | 直接改值即可  |
| 查  | 按索引随机访问  | 靠指针依次访问  |

> 关键区别：存储和查询

## 遍历访问

> 分为两种：线性迭代访问，非线性递归访问

### 数组

```go
var arr = []string{"a", "b", "c", "d"}

func visitArrIter(arr []string) {
	for i := 0; i < len(arr); i++ {
		fmt.Println(arr[i])	// 访问
	}
}

func visitArrRec(arr []string, index int)  {
	if index >= len(arr) {  	//基
		return
	}

	fmt.Println(arr[index]) 	//访问
	visitArrRec(arr, index+1)  	//递归
}
```

### 链表

```go
type listNode struct {
	val string
	next *listNode
}

func visitListIter(head *listNode) {
	for p := head; p != nil; p = p.next {
		fmt.Println(p.val)	//访问
	}
}

func visitListRec(head *listNode) {
	if head == nil {			//基
		return
	}

	fmt.Println(head.val)		//访问
	visitListRec(head.next)		//递归
}
```

## 参考

- https://labuladong.gitbook.io/algo/di-ling-zhang-bi-du-xi-lie/xue-xi-shu-ju-jie-gou-he-suan-fa-de-gao-xiao-fang-fa
