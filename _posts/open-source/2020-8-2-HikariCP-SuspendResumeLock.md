---
title: HikariCP：限流并发控制SuspendResumeLock
date: 2020-8-2 17:40:52
tags: [HikariCP]
---

com.zaxxer.hikari.util.SuspendResumeLock

## 无处理

```java
   public static final SuspendResumeLock FAUX_LOCK = new SuspendResumeLock(false) {
      @Override
      public void acquire() {}

      @Override
      public void release() {}

      @Override
      public void suspend() {}

      @Override
      public void resume() {}
   };
```

> 根据isAllowPoolSuspension标识，判断是否需要进行并发处理

## 并发控制

```java
   private static final int MAX_PERMITS = 10000;
   private final Semaphore acquisitionSemaphore;
```

- 默认并发数：10000
- 通过对Semaphore封装处理


```java
   public SuspendResumeLock()
   {
      this(true);
   }

   private SuspendResumeLock(final boolean createSemaphore)
   {
      //公平锁，先到先得
      acquisitionSemaphore = (createSemaphore ? new Semaphore(MAX_PERMITS, true) : null);
   }

   public void acquire() throws SQLException
   {
      if (acquisitionSemaphore.tryAcquire()) {
         return;
      }
      else if (Boolean.getBoolean("com.zaxxer.hikari.throwIfSuspended")) {
         throw new SQLTransientException("The pool is currently suspended and configured to throw exceptions upon acquisition");
      }

      acquisitionSemaphore.acquireUninterruptibly();
   }

   public void release()
   {
      acquisitionSemaphore.release();
   }

   public void suspend()
   {
      //获取所有锁，模拟挂起
      acquisitionSemaphore.acquireUninterruptibly(MAX_PERMITS);
   }

   public void resume()
   {
      acquisitionSemaphore.release(MAX_PERMITS);
   }
```

- 公平锁，先到先得

## 使用

```java
   public Connection getConnection(final long hardTimeout) throws SQLException
   {
      suspendResumeLock.acquire();
      final long startTime = currentTime();

      try {
......
      finally {
         suspendResumeLock.release();
      }
   }
```

> 通过suspendResumeLock，控制获取数据库连接动作getConnection的并发访问
