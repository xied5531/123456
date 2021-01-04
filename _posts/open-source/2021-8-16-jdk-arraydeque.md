---
title: JDK：ArrayDeque分析
date: 2020-8-16 10:02:06
tags: JDK
---

# JDK：ArrayDeque分析

## 数据结构

```java
    // 采用Object数组做存储介质，自定义序列化方法
    transient Object[] elements;

    // 首元素索引
    transient int head;

    // 尾元素索引
    transient int tail;

    // 最大大小值，预留了8个元素空间
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

## 初始化

```java
    // 默认预留16个元素空间
    public ArrayDeque() {
        elements = new Object[16 + 1];
    }
    // 最小预留1个，最大预留Integer.MAX_VALUE个，正常预留+1个元素空间
    public ArrayDeque(int numElements) {
        elements =
            new Object[(numElements < 1) ? 1 :
                       (numElements == Integer.MAX_VALUE) ? Integer.MAX_VALUE :
                       numElements + 1];
    }

    // 按序复制集合元素
    public ArrayDeque(Collection<? extends E> c) {
        this(c.size());
        copyElements(c);
    }
```

## 出入队

```java
    // 头插
    public void addFirst(E e) {
        if (e == null)
            throw new NullPointerException();
        final Object[] es = elements;
        // 计算新head索引，然后判断是否扩容
        es[head = dec(head, es.length)] = e;
        if (head == tail)
            grow(1);
    }
    // 尾插
    public void addLast(E e) {
        if (e == null)
            throw new NullPointerException();
        final Object[] es = elements;
        es[tail] = e;
        // 计算新tail索引，然后判断是否扩容
        if (head == (tail = inc(tail, es.length)))
            grow(1);
    }
```



```java
    // 头删
    public E pollFirst() {
        final Object[] es;
        final int h;
        E e = elementAt(es = elements, h = head);
        if (e != null) {
            es[h] = null;
            // 更新head索引
            head = inc(h, es.length);
        }
        return e;
    }
    // 尾删
    public E pollLast() {
        final Object[] es;
        final int t;
        E e = elementAt(es = elements, t = dec(tail, es.length));
        if (e != null)
            // 更新tail索引
            es[tail = t] = null;
        return e;
    }
```

索引模型（tail在前，head在后）

```
0 1 2 3 . . . len-4 len-3 len-2 len-1
tail ............................head
```

### 索引计算

```java
    // 统一定义：0 <= i < modulus
    // (i+1)%modulus
    static final int inc(int i, int modulus) {
        if (++i >= modulus) i = 0; // 溢出从0开始
        return i;
    }

    // (i-1)%modulus
    static final int dec(int i, int modulus) {
        if (--i < 0) i = modulus - 1; // 溢出从modulus - 1开始
        return i;
    }

    // (i+distance)%modulus
    static final int inc(int i, int distance, int modulus) {
        if ((i += distance) - modulus >= 0) i -= modulus; // 溢出从头开始算
        return i;
    }

    // (i-j)%modulus
    static final int sub(int i, int j, int modulus) {
        if ((i -= j) < 0) i += modulus; // 溢出从尾开始算
        return i;
    }
```

## 扩容

```java
    // 新增needed个元素空间
    private void grow(int needed) {
        // overflow-conscious code
        final int oldCapacity = elements.length;
        int newCapacity;
        // Double capacity if small; else grow by 50%
        // 容量<64时，扩容一倍，否则扩容一半
        int jump = (oldCapacity < 64) ? (oldCapacity + 2) : (oldCapacity >> 1);
        if (jump < needed
            || (newCapacity = (oldCapacity + jump)) - MAX_ARRAY_SIZE > 0)
            // 默认扩容不足或扩容后容量过大后，进行特殊处理
            newCapacity = newCapacity(needed, jump);
        final Object[] es = elements = Arrays.copyOf(elements, newCapacity);
        // Exceptionally, here tail == head needs to be disambiguated
        if (tail < head || (tail == head && es[head] != null)) {
            // wrap around; slide first leg forward to end of array
            int newSpace = newCapacity - oldCapacity;
            // 扩容后，移动es[head,oldCapacity]至es[head + newSpace,newCapacity]处
            System.arraycopy(es, head,
                             es, head + newSpace,
                             oldCapacity - head);
            // 将es[head,head+newSpace]元素赋null
            for (int i = head, to = (head += newSpace); i < to; i++)
                es[i] = null;
        }
    }

    /** Capacity calculation for edge conditions, especially overflow. */
    private int newCapacity(int needed, int jump) {
        final int oldCapacity = elements.length, minCapacity;
        if ((minCapacity = oldCapacity + needed) - MAX_ARRAY_SIZE > 0) {
            if (minCapacity < 0)
                // 溢出抛异常
                throw new IllegalStateException("Sorry, deque too big");
            return Integer.MAX_VALUE;
        }
        if (needed > jump)
            return minCapacity;
        return (oldCapacity + jump - MAX_ARRAY_SIZE < 0)
            ? oldCapacity + jump
            : MAX_ARRAY_SIZE;
    }
```

