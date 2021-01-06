---
title: JDK：HashSet分析
date: 2020-8-23 10:02:06
tags: JDK
---

# JDK：HashSet分析

## 数据结构

```java
    private transient HashMap<E,Object> map;

    // 占位元素
    private static final Object PRESENT = new Object();
```

说明：基于HashMap实现

## 初始化

```java
    public HashSet() {
        map = new HashMap<>();
    }

    public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }

    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }

    public HashSet(int initialCapacity) {
        map = new HashMap<>(initialCapacity);
    }
```

## CRUD

```java
    public boolean contains(Object o) {
        return map.containsKey(o);
    }

    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }

    public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
    }
```

## 总结

- 基于HashMap实现
