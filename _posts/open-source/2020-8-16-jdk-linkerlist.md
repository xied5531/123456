---
title: JDK：LinkedList分析
date: 2020-8-16 11:02:06
tags: JDK
---

# JDK：LinkedList分析

## Node

```java
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

前后节点双向指针

初始化时指定前后节点

## 属性

```java
    transient int size = 0;

    /**
     * Pointer to first node.
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     */
    transient Node<E> last;
```

## 初始化

```java
    /**
     * Constructs an empty list.
     */
    public LinkedList() {
    }
```

## 增加

```java
    /**
     * Links e as first element.
     */
    private void linkFirst(E e) {
        final Node<E> f = first; // 暂存头指针
        final Node<E> newNode = new Node<>(null, e, f); // 新建节点并指向头
        first = newNode; // 更新头
        if (f == null)
            last = newNode; // 更新尾
        else
            f.prev = newNode; // 更新前驱
        size++;
        modCount++;
    }

    /**
     * Links e as last element.
     */
    void linkLast(E e) {
        final Node<E> l = last; // 暂存尾指针
        final Node<E> newNode = new Node<>(l, e, null); // 新建节点并对接尾
        last = newNode; // 更新尾
        if (l == null)
            first = newNode; // 更新头
        else
            l.next = newNode; // 更新后继
        size++;
        modCount++;
    }

    /**
     * Inserts element e before non-null Node succ.
     */
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev; // 暂存前驱
        final Node<E> newNode = new Node<>(pred, e, succ); // 新建节点并链接前后
        succ.prev = newNode; // 更新前驱
        if (pred == null)
            first = newNode; // 更新头
        else
            pred.next = newNode; // 更新后继
        size++;
        modCount++;
    }
```

暂存当前节点及前后指针

特殊处理头和尾

## 删除

```java
    /**
     * Unlinks non-null first node f.
     */
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next; // 获取后继
        f.item = null;
        f.next = null; // help GC
        first = next; // 更新头
        if (next == null)
            last = null; // 更新尾
        else
            next.prev = null; // 更新前驱
        size--;
        modCount++;
        return element;
    }

    /**
     * Unlinks non-null last node l.
     */
    private E unlinkLast(Node<E> l) {
        // assert l == last && l != null;
        final E element = l.item;
        final Node<E> prev = l.prev; // 获取前驱
        l.item = null;
        l.prev = null; // help GC
        last = prev; // 更新尾
        if (prev == null)
            first = null; // 更新头
        else
            prev.next = null; // 更新后继
        size--;
        modCount++;
        return element;
    }

    /**
     * Unlinks non-null node x.
     */
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next; // 获取后继
        final Node<E> prev = x.prev; // 获取前驱

        if (prev == null) {
            first = next; // 更新头
        } else {
            prev.next = next; // 更新后继
            x.prev = null; // 中断前驱
        }

        if (next == null) {
            last = prev; // 更新尾
        } else {
            next.prev = prev; // 更新前驱
            x.next = null; // 中断后继
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }

```

暂存当前节点及前后指针

特殊处理头和尾

中断旧链接

## 查找

```java
    // 正向查找，从first开始
    public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }

    // 逆向查找，从last开始
    public int lastIndexOf(Object o) {
        int index = size;
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (x.item == null)
                    return index;
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (o.equals(x.item))
                    return index;
            }
        }
        return -1;
    }
```

