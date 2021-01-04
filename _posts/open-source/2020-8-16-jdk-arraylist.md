---
title: JDK：ArrayList分析
date: 2020-8-16 10:02:06
tags: JDK
---

# JDK：ArrayList分析

## 数据结构

```java
    /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * Shared empty array instance used for empty instances.
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     */
    private int size;
```

大小：

1. capacity，容量，实际占用的大小，非空列表，默认初始大小为10
2. size，大小，实际使用的大小

介质：

1. `transient Object[] elementData`，Object数组存储任意对象，非自动序列化，通过`readObject`和`writeObject`主动实现序列化
2. `EMPTY_ELEMENTDATA`，初始空列表，指定容量为0时
3. `DEFAULTCAPACITY_EMPTY_ELEMENTDATA`，初始空列表，未指定容量时

## 初始化

```java
    // 指定容量为0时，直接引用EMPTY_ELEMENTDATA
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
    // 默认直接引用DEFAULTCAPACITY_EMPTY_ELEMENTDATA
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
    // 复制非空Collection内容，为空时直接引用EMPTY_ELEMENTDATA
    public ArrayList(Collection<? extends E> c) {
        Object[] a = c.toArray();
        if ((size = a.length) != 0) {
            if (c.getClass() == ArrayList.class) {
                elementData = a;
            } else {
                elementData = Arrays.copyOf(a, size, Object[].class);
            }
        } else {
            // replace with empty array.
            elementData = EMPTY_ELEMENTDATA;
        }
    }
```

## 扩减容

减容

```java
    public void trimToSize() {
        modCount++;
        // 如果实际使用大小 < 实际占用大小
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA //为空时，直接引用EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size); //非空时，复制至新数组
        }
    }
```

扩容

```java
    public void ensureCapacity(int minCapacity) {
        // 如果目标大小 > 实际占用大小 并且 > 默认大小
        if (minCapacity > elementData.length
            && !(elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
                 && minCapacity <= DEFAULT_CAPACITY)) {
            modCount++;
            grow(minCapacity);
        }
    }

    // 扩容至目标大小minCapacity
    private Object[] grow(int minCapacity) {
        int oldCapacity = elementData.length;
        // 非首次扩容时
        if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            int newCapacity = ArraysSupport.newLength(oldCapacity,
                    minCapacity - oldCapacity, /* minimum growth */
                    oldCapacity >> 1           /* preferred growth */);
            return elementData = Arrays.copyOf(elementData, newCapacity);
        } else {
            // 首次扩容时，直接扩容至最小默认值
            return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
        }
    }

	//默认扩容一个单位
    private Object[] grow() {
        return grow(size + 1);
    }
```

## 搜索

正向

```java
    public int indexOf(Object o) {
        return indexOfRange(o, 0, size);
    }

	// 从前往后搜
    int indexOfRange(Object o, int start, int end) {
        Object[] es = elementData;
        // 目标为null时
        if (o == null) {
            for (int i = start; i < end; i++) {
                if (es[i] == null) {
                    return i;
                }
            }
        } else {
            // 目标非null时
            for (int i = start; i < end; i++) {
                if (o.equals(es[i])) {
                    return i;
                }
            }
        }
        return -1;
    }
```

```java
    public int lastIndexOf(Object o) {
        return lastIndexOfRange(o, 0, size);
    }
	// 从后往前搜
    int lastIndexOfRange(Object o, int start, int end) {
        Object[] es = elementData;
        // 目标为null时
        if (o == null) {
            for (int i = end - 1; i >= start; i--) {
                if (es[i] == null) {
                    return i;
                }
            }
        } else {
            // 目标非null时
            for (int i = end - 1; i >= start; i--) {
                if (o.equals(es[i])) {
                    return i;
                }
            }
        }
        return -1;
    }
```

## CRUD

get和set

```java
    // 直接取值
    public E get(int index) {
        Objects.checkIndex(index, size);
        return elementData(index);
    }
    // 直接赋值
    public E set(int index, E element) {
        Objects.checkIndex(index, size);
        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
```

add和remove

```java
    private void add(E e, Object[] elementData, int s) {
        // 满了的话，需要先扩容
        if (s == elementData.length)
            elementData = grow();
        elementData[s] = e;
        size = s + 1;
    }
    public boolean add(E e) {
        modCount++;
        add(e, elementData, size);
        return true;
    }
	// 中间位置插入
	public void add(int index, E element) {
        rangeCheckForAdd(index);
        modCount++;
        final int s;
        Object[] elementData;
        // 获取临时size和elementData，并判断是否满了，然后扩容
        if ((s = size) == (elementData = this.elementData).length)
            elementData = grow();
        // 直接系统复制
        System.arraycopy(elementData, index,
                         elementData, index + 1,
                         s - index);
        elementData[index] = element;
        size = s + 1;
    }
```

```java
    // 指定位置删除
    public E remove(int index) {
        Objects.checkIndex(index, size);
        final Object[] es = elementData;

        @SuppressWarnings("unchecked") E oldValue = (E) es[index];
        // 获取临时elementData和待删除的元素oldValue
        fastRemove(es, index);

        return oldValue;
    }
	// 指定元素删除
    public boolean remove(Object o) {
        final Object[] es = elementData;
        final int size = this.size;
        int i = 0;
        // 搜索删除位置
        found: {
            if (o == null) {
                for (; i < size; i++)
                    if (es[i] == null)
                        break found;
            } else {
                for (; i < size; i++)
                    if (o.equals(es[i]))
                        break found;
            }
            return false;
        }
        fastRemove(es, i);
        return true;
    }

    private void fastRemove(Object[] es, int i) {
        modCount++;
        final int newSize;
        // 非最后位置删除时，执行系统复制
        if ((newSize = size - 1) > i)
            System.arraycopy(es, i + 1, es, i, newSize - i);
        // 最后元素置null
        es[size = newSize] = null;
    }
```

## 序列化

```java
    // 写对象
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // 只写实际使用的大小
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }
		// 过程中，如果列表被修改则抛异常
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }

    // 读对象
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {

        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in capacity
        s.readInt(); // ignored

        if (size > 0) {
            // like clone(), allocate array based upon size not capacity
            SharedSecrets.getJavaObjectInputStreamAccess().checkArray(s, Object[].class, size);
            Object[] elements = new Object[size];

            // Read in all elements in the proper order.
            for (int i = 0; i < size; i++) {
                elements[i] = s.readObject();
            }

            elementData = elements;
        } else if (size == 0) {
            elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new java.io.InvalidObjectException("Invalid size: " + size);
        }
    }
```

说明

1. 序列化对象，读写内容仅限实际使用的大小，避免多余数据的传输
2. 过程中，如果modCount被修改，则抛异常，实现fail-fast机制，即不支持并发更新操作

## 迭代器

```java
    private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        // prevent creating a synthetic constructor
        Itr() {}

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        public void forEachRemaining(Consumer<? super E> action) {
            Objects.requireNonNull(action);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i < size) {
                final Object[] es = elementData;
                if (i >= es.length)
                    throw new ConcurrentModificationException();
                for (; i < size && modCount == expectedModCount; i++)
                    action.accept(elementAt(es, i));
                // update once at end to reduce heap write traffic
                cursor = i;
                lastRet = i - 1;
                checkForComodification();
            }
        }
		// 迭代过程中，不允许对列表进行修改操作
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

```

```java
    // 优化版迭代器，迭代过程中允许add和set操作
	private class ListItr extends Itr implements ListIterator<E> {
        ListItr(int index) {
            super();
            cursor = index;
        }

        public boolean hasPrevious() {
            return cursor != 0;
        }

        public int nextIndex() {
            return cursor;
        }

        public int previousIndex() {
            return cursor - 1;
        }

        @SuppressWarnings("unchecked")
        public E previous() {
            checkForComodification();
            int i = cursor - 1;
            if (i < 0)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i;
            return (E) elementData[lastRet = i];
        }

        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.set(lastRet, e);
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        public void add(E e) {
            checkForComodification();

            try {
                int i = cursor;
                ArrayList.this.add(i, e);
                cursor = i + 1;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
    }
```

