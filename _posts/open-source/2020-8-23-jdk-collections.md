---
title: JDK：Collections分析
date: 2020-8-23 12:02:06
tags: JDK
---

# JDK：Collections分析

## binarySearch

```java
    private static <T>
    int indexedBinarySearch(List<? extends Comparable<? super T>> list, T key) {
        int low = 0;
        int high = list.size()-1;

        while (low <= high) {
            int mid = (low + high) >>> 1;
            Comparable<? super T> midVal = list.get(mid);
            int cmp = midVal.compareTo(key);

            if (cmp < 0)
                low = mid + 1;
            else if (cmp > 0)
                high = mid - 1;
            else
                return mid; // key found
        }
        return -(low + 1);  // key not found
    }
```

标准二分查找算法

## reverse

```java
    public static void reverse(List<?> list) {
        int size = list.size();
        if (size < REVERSE_THRESHOLD || list instanceof RandomAccess) {
            for (int i=0, mid=size>>1, j=size-1; i<mid; i++, j--)
                swap(list, i, j);
        } else {
            ListIterator fwd = list.listIterator();
            ListIterator rev = list.listIterator(size);
            for (int i=0, mid=list.size()>>1; i<mid; i++) {
                Object tmp = fwd.next();
                fwd.set(rev.previous());
                rev.set(tmp);
            }
        }
    }
```

前后缩减并交换值，中间值固定

## rotate

```java
    private static <T> void rotate1(List<T> list, int distance) {
        int size = list.size();
        if (size == 0)
            return;
        // 找目标点
        distance = distance % size;
        if (distance < 0)
            distance += size;
        if (distance == 0)
            return;

        for (int cycleStart = 0, nMoved = 0; nMoved != size; cycleStart++) {
            // 从第一个位置开始
            T displaced = list.get(cycleStart);
            int i = cycleStart;
            do {
                i += distance;
                if (i >= size)
                    i -= size;
                // 将当前位置的元素放至目标位置，
                // 当前目标位置的元素为错位元素，下轮循环将其放入正确位置
                displaced = list.set(i, displaced);
                nMoved ++; // 每放至一次目标位置，进行计数
            } while (i != cycleStart); // 当错误位置的元素刚好需要放至到当前位置时结束本轮循环
            // 直到计数等于总数时相当于所有元素均移动完毕
        }
    }
```

所有元素一步到位

```java
    private static void rotate2(List<?> list, int distance) {
        int size = list.size();
        if (size == 0)
            return;
        // 找中间点，默认正向旋转
        int mid =  -distance % size;
        if (mid < 0)
            mid += size;
        if (mid == 0)
            return;
        // 分段逆序
        reverse(list.subList(0, mid));
        reverse(list.subList(mid, size));
        // 整体逆序
        reverse(list);
    }
```

双逆序+整体逆序=所有元素就位
