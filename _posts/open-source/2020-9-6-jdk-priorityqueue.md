---
title: JDK：PriorityQueue分析
date: 2020-9-6 08:02:06
tags: JDK
---

# JDK：PriorityQueue分析

## 数据结构

```java
    // 默认占用空间大小11
    private static final int DEFAULT_INITIAL_CAPACITY = 11;
    // 基于数组实现
    transient Object[] queue; // non-private to simplify nested class access
    // 使用大小
    int size;
```

## 扩容

```java
    private void grow(int minCapacity) {
        int oldCapacity = queue.length;
        // Double size if small; else grow by 50%
        int newCapacity = ArraysSupport.newLength(oldCapacity,
                minCapacity - oldCapacity, /* minimum growth */
                oldCapacity < 64 ? oldCapacity + 2 : oldCapacity >> 1
                                           /* preferred growth */);
        queue = Arrays.copyOf(queue, newCapacity);
    }
```

小于64时扩容1倍，否则扩容1半

## CRUD

插入

```java
    public boolean offer(E e) {
        if (e == null)
            throw new NullPointerException();
        modCount++;
        int i = size;
        if (i >= queue.length)
            grow(i + 1); // 满则先扩容
        siftUp(i, e); // 先放入最后，然后上浮至正确位置
        size = i + 1;
        return true;
    }
```

查询

```java
    public E peek() {
        return (E) queue[0]; // 默认取第一个元素
    }
```

删除

```java
    E removeAt(int i) {
        // assert i >= 0 && i < size;
        final Object[] es = queue;
        modCount++;
        int s = --size;
        if (s == i) // removed last element
            es[i] = null; // 如果刚好是最后元素，直接赋值null
        else {
            E moved = (E) es[s];
            es[s] = null; // 删除最后元素并获取对应值
            siftDown(i, moved); // 将最后元素赋值至待删除位置，下沉至正确位置
            if (es[i] == moved) { // 如果没有下沉，则进行上浮
                siftUp(i, moved);
                if (es[i] != moved)
                    return moved;
            }
        }
        return null;
    }
```

## 下沉和上浮

```java
    // 上浮
    private static <T> void siftUpComparable(int k, T x, Object[] es) {
        Comparable<? super T> key = (Comparable<? super T>) x;
        while (k > 0) {
            int parent = (k - 1) >>> 1; // 找到父节点
            Object e = es[parent];
            if (key.compareTo((T) e) >= 0)
                break;
            es[k] = e; // 用父节点替换
            k = parent; // 接着从父节点开始循环
        }
        es[k] = key; // 将key放至正确位置
    }
    // 下沉
    private static <T> void siftDownComparable(int k, T x, Object[] es, int n) {
        // assert n > 0;
        Comparable<? super T> key = (Comparable<? super T>)x;
        int half = n >>> 1;           // loop while a non-leaf
        while (k < half) {
            int child = (k << 1) + 1; // assume left child is least
            Object c = es[child];
            int right = child + 1;
            if (right < n &&
                ((Comparable<? super T>) c).compareTo((T) es[right]) > 0)
                c = es[child = right]; // 找到左右子节点合适者
            if (key.compareTo((T) c) <= 0)
                break;
            es[k] = c; // 用子节点替换
            k = child; // 接着从子节点开始找
        }
        es[k] = key; // 将key放至正确位置
    }
```

