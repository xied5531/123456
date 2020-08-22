---
title: HikariCP：数据库连接资源池管理ConcurrentBag
date: 2020-8-2 14:41:52
tags: [HikariCP]
---

## 源码

https://github.com/xied5531/src-HikariCP-3.4.5/blob/master/src/main/java/com/zaxxer/hikari/util/ConcurrentBag.java

数据库连接资源池管理，资源元素为对数据库连接的封装对象，提供增加、删除、借用、归还的核心功能

## 资源元素

```java
public interface IConcurrentBagEntry
   {
      int STATE_NOT_IN_USE = 0;
      int STATE_IN_USE = 1;
      int STATE_REMOVED = -1;
      int STATE_RESERVED = -2;

      boolean compareAndSet(int expectState, int newState);
      void setState(int newState);
      int getState();
   }
```

1. 统一接口，默认存放对数据库连接封装的PoolEntry对象
2. 支持四种状态，未使用、使用中、删除、保留

## 数据结构

```java
   private final CopyOnWriteArrayList<T> sharedList;

   private final ThreadLocal<List<Object>> threadList;

   private final AtomicInteger waiters;

   private final SynchronousQueue<T> handoffQueue;
```

1. CopyOnWriteArrayList<T> sharedList，作为后端存储介质
2. ThreadLocal<List<Object>> threadList，线程级缓存资源List
3. SynchronousQueue<T> handoffQueue，无实际介质的后端存储介质，提高数据库连接在线程间的复用速度
4. AtomicInteger waiters，等候线程计算器

```java
   public interface IBagStateListener
   {
      void addBagItem(int waiting);
   }
```

外部钩子，支持创建数据库连接

## 核心操作

### add，新增资源

```java
   public void add(final T bagEntry)
   {
      //是否关闭
      if (closed) {
         LOGGER.info("ConcurrentBag has been closed, ignoring add()");
         throw new IllegalStateException("ConcurrentBag has been closed, ignoring add()");
      }
      //添加至后端存储介质
      sharedList.add(bagEntry);

      //如果有等候人，并且改连接未被使用，发送至无介质后端存储，快速让等待人复用
      while (waiters.get() > 0 && bagEntry.getState() == STATE_NOT_IN_USE && !handoffQueue.offer(bagEntry)) {
         Thread.yield();
      }
   }
```

关键点：先添加至后端存储，然后如果线程再等候直接转交

### remove，删除资源

```java
   public boolean remove(final T bagEntry)
   {
      //如果未使用或未保留，标记删除状态，否则进行物理删除
      if (!bagEntry.compareAndSet(STATE_IN_USE, STATE_REMOVED) && !bagEntry.compareAndSet(STATE_RESERVED, STATE_REMOVED) && !closed) {
         LOGGER.warn("Attempt to remove an object from the bag that was not borrowed or reserved: {}", bagEntry);
         return false;
      }

      //从后端存储介质删除
      final boolean removed = sharedList.remove(bagEntry);
      if (!removed && !closed) {
         LOGGER.warn("Attempt to remove an object from the bag that does not exist: {}", bagEntry);
      }

      //从当前缓存中删除
      threadList.get().remove(bagEntry);

      return removed;
   }
```

关键点：如果是使用中或保留的仅标记删除，否则进行物理删除，并清除当前线程缓存

### borrow，借用资源

```java
   public T borrow(long timeout, final TimeUnit timeUnit) throws InterruptedException
   {
      //检查当前线程缓存资源
      final List<Object> list = threadList.get();
      //从后往前检查
      for (int i = list.size() - 1; i >= 0; i--) {
         final Object entry = list.remove(i);
         @SuppressWarnings("unchecked")
         final T bagEntry = weakThreadLocals ? ((WeakReference<T>) entry).get() : (T) entry;
         //已存在且未被使用
         if (bagEntry != null && bagEntry.compareAndSet(STATE_NOT_IN_USE, STATE_IN_USE)) {
            return bagEntry;
         }
      }

      //等候人++
      final int waiting = waiters.incrementAndGet();
      try {
         //检查后端存储资源List
         for (T bagEntry : sharedList) {
            //获取到未在使用中的资源
            if (bagEntry.compareAndSet(STATE_NOT_IN_USE, STATE_IN_USE)) {
               //如果还有等候人，调用资源创建钩子函数
               if (waiting > 1) {
                  listener.addBagItem(waiting - 1);
               }
               return bagEntry;
            }
         }

         //如果无可用资源，调用资源创建钩子函数
         listener.addBagItem(waiting);

         //超时
         timeout = timeUnit.toNanos(timeout);
         do {
            //开始时间
            final long start = currentTime();
            //从无介质的后端存储中检查是否有可用资源
            final T bagEntry = handoffQueue.poll(timeout, NANOSECONDS);
            //获取到未使用中资源
            if (bagEntry == null || bagEntry.compareAndSet(STATE_NOT_IN_USE, STATE_IN_USE)) {
               return bagEntry;
            }
            //超时时间更新
            timeout -= elapsedNanos(start);
         } while (timeout > 10_000);

         return null;
      }
      finally {
         //等候人--
         waiters.decrementAndGet();
      }
   }
```

关键点：先从当前线程缓存中检查是否有可用资源，然后从后端存储中检查并通知钩子函数，最后通知钩子函数还是现等可用的资源

### require，归还资源

```java
   public void requite(final T bagEntry)
   {
      //更新状态至未使用
      bagEntry.setState(STATE_NOT_IN_USE);

      //遍历可用等候人
      for (int i = 0; waiters.get() > 0; i++) {
         //是否，已被使用或转发成功
         if (bagEntry.getState() != STATE_NOT_IN_USE || handoffQueue.offer(bagEntry)) {
            return;
         }
         //数量太多，禁用一会
         else if ((i & 0xff) == 0xff) {
            parkNanos(MICROSECONDS.toNanos(10));
         }
         //给其他线程让路
         else {
            Thread.yield();
         }
      }

      //没有人接收，放入当前线程的缓存中，最大50个
      final List<Object> threadLocalList = threadList.get();
      if (threadLocalList.size() < 50) {
         threadLocalList.add(weakThreadLocals ? new WeakReference<>(bagEntry) : bagEntry);
      }
   }
```

关键点：先检查是否有现场等待的线程，然后放入本地缓存中
