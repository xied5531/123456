---
title: 递归：力扣206. 反转链表
date: 2020-9-6 0:33:54
tags: 递归
---

# 递归：力扣206. 反转链表

## 206. 反转链表

反转一个单链表。

示例:

```
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

## 最简场景

只有一个节点或没有节点时，例如：`NULL`和`5->NULL`

直接返回当前节点即可

```go
    if head == nil || head.Next == nil {
        return head
    }
```

## 递归实施

### 缩小规模，减治或分治

前提：缩小规模后的问题求解和当前规模的问题求解过程是同质的

1. 当前规模：反转以1为head的链表
2. 缩小规模：反转以1的后继2为head的链表

### 确定返回值

链表反转后的head节点，例如：以2为head的链表

反转前：`2->3->4->5->NULL`

反转后：`NULL<-2<-3<-4<-5`

返回值为5的head节点

> 注意：当前1的后继仍然为2，即`1->2`

```
    newHead := reverseList(head.Next)
```

### 整合结果

一轮递归后当前状态：当前head为1，newHead为5

```
   NULL
   ^
   |
1->2<-3<-4<-5
```

目标：将当前状态调整为满足递归函数定义的状态

- 让2的后继指向1。由于只知道当前head为1，head.Next即为2，通过当前head进行调整：`head.Next.Next = head`

```
1<->2<-3<-4<-5
```

- 让1的后继指向NULL，即头变尾，完整角色反转：`head.Next = nil`

```
NULL<-1<-2<-3<-4<-5
```

- 让newHead作为当前的head：`return newHead`

```
5->4->3->2->1->NULL
```

完成本轮链表反转

## 代码

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reverseList(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }

    newHead := reverseList(head.Next)

    head.Next.Next = head
    head.Next = nil
    return newHead
}
```
