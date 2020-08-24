---
title: HikariCP：连接资源池管理HikariPool
date: 2020-8-2 16:40:52
tags: [HikariCP]
---

https://github.com/xied5531/src-HikariCP-3.4.5/blob/master/src/main/java/com/zaxxer/hikari/pool/HikariPool.java

## 属性

状态：

```java
   public static final int POOL_NORMAL = 0;
   public static final int POOL_SUSPENDED = 1;
   public static final int POOL_SHUTDOWN = 2;

   public volatile int poolState;
```

> 正常、挂起、关闭

任务：

```java
   private final ThreadPoolExecutor addConnectionExecutor;	//异步创建数据库连接
   private final ThreadPoolExecutor closeConnectionExecutor;	//异步关闭数据库连接
   
   private final ScheduledExecutorService houseKeepingExecutorService;	//定时控制数据库连接总数
```

并发：

```java
   private final SuspendResumeLock suspendResumeLock;
```

> isAllowPoolSuspension，如果开启，获取连接操作，默认最大并发数10000，否则不限流

存储：

```java
   private final ConcurrentBag<PoolEntry> connectionBag;
```

## 获取数据库连接，getConnection

```java
   public Connection getConnection(final long hardTimeout) throws SQLException
   {
      //获取锁
      suspendResumeLock.acquire();
      final long startTime = currentTime();

      try {
         long timeout = hardTimeout;
         do {
            //借用连接
            PoolEntry poolEntry = connectionBag.borrow(timeout, MILLISECONDS);
            if (poolEntry == null) {
               break; //未借到，抛异常
            }

            final long now = currentTime();
            //连接被驱逐，或超时并且非活着状态，则关闭连接
            if (poolEntry.isMarkedEvicted() || (elapsedMillis(poolEntry.lastAccessed, now) > aliveBypassWindowMs && !isConnectionAlive(poolEntry.connection))) {
               closeConnection(poolEntry, poolEntry.isMarkedEvicted() ? EVICTED_CONNECTION_MESSAGE : DEAD_CONNECTION_MESSAGE);
               timeout = hardTimeout - elapsedMillis(startTime);
            }
            else {
               //记录信息
               metricsTracker.recordBorrowStats(poolEntry, startTime);
               //创建代理连接，并开启资源泄露检测任务
               return poolEntry.createProxyConnection(leakTaskFactory.schedule(poolEntry), now);
            }
         } while (timeout > 0L);

         metricsTracker.recordBorrowTimeoutStats(startTime);
         throw createTimeoutException(startTime);
      }
      catch (InterruptedException e) {
         Thread.currentThread().interrupt();
         throw new SQLException(poolName + " - Interrupted during connection acquisition", e);
      }
      finally {
         //释放锁
         suspendResumeLock.release();
      }
   }
```

1. 通过connectionBag.borrow借用数据库连接对象PoolEntry
2. 检查PoolEntry可用性
3. 通过poolEntry.createProxyConnection获取数据库连接的ProxyConnection，并设置资源泄露检测任务

## 创建数据库连接，addBagItem

```java
   @Override
   public void addBagItem(final int waiting)
   {
      //等候数>创建队列时
      final boolean shouldAdd = waiting - addConnectionQueueReadOnlyView.size() >= 0; // Yes, >= is intentional.
      if (shouldAdd) {
         //新建创建连接任务
         addConnectionExecutor.submit(poolEntryCreator);
      }
      else {
         logger.debug("{} - Add connection elided, waiting {}, queue {}", poolName, waiting, addConnectionQueueReadOnlyView.size());
      }
   }
```

> 实现com.zaxxer.hikari.util.ConcurrentBag.IBagStateListener接口，作为ConcurrentBag钩子函数

```java
   public interface IBagStateListener
   {
      void addBagItem(int waiting);
   }
```

## 异步任务

### 数据库连接对象创建，PoolEntryCreator 

```java
   private final class PoolEntryCreator implements Callable<Boolean>
   {
      private final String loggingPrefix;

      //设置任务名
      PoolEntryCreator(String loggingPrefix)
      {
         this.loggingPrefix = loggingPrefix;
      }

      @Override
      public Boolean call()
      {
         long sleepBackoff = 250L;
		 //检查当前池子状态，判断是否需要创建对象
         while (poolState == POOL_NORMAL && shouldCreateAnotherConnection()) {
            final PoolEntry poolEntry = createPoolEntry();
            if (poolEntry != null) {
               connectionBag.add(poolEntry);
               logger.debug("{} - Added connection {}", poolName, poolEntry.connection);
               if (loggingPrefix != null) {
                  logPoolState(loggingPrefix);
               }
               return Boolean.TRUE;
            }

            //创建对象失败，休息250ms后，重试
            if (loggingPrefix != null) logger.debug("{} - Connection add failed, sleeping with backoff: {}ms", poolName, sleepBackoff);
            quietlySleep(sleepBackoff);
			//调整休息时间间隔，自动增加间隔，最大间隔10秒
            sleepBackoff = Math.min(SECONDS.toMillis(10), Math.min(connectionTimeout, (long) (sleepBackoff * 1.5)));
         }

         //创建失败
         return Boolean.FALSE;
      }

      //判断条件：总连接数 < 最大连接数，并且，（空闲连接数 < 最小空闲数据，或者，有等候线程）
      private synchronized boolean shouldCreateAnotherConnection() {
         return getTotalConnections() < config.getMaximumPoolSize() &&
            (connectionBag.getWaitingThreadCount() > 0 || getIdleConnections() < config.getMinimumIdle());
      }
   }
```

1. 检查当前池子状态，判断是否需要创建对象
2. 创建数据库连接对象
3. 创建对象失败，休息间隔后（自动增加间隔，最大间隔10秒），然后重试

```java
   private PoolEntry createPoolEntry()
   {
      try {
         //新建对象
         final PoolEntry poolEntry = newPoolEntry();

         final long maxLifetime = config.getMaxLifetime();
         if (maxLifetime > 0) {
            // variance up to 2.5% of the maxlifetime
            // 计算生命时间
            final long variance = maxLifetime > 10_000 ? ThreadLocalRandom.current().nextLong( maxLifetime / 40 ) : 0;
            final long lifetime = maxLifetime - variance;
            // 设置超时控制任务
            poolEntry.setFutureEol(houseKeepingExecutorService.schedule(
               () -> {
                  // 软驱逐
                  if (softEvictConnection(poolEntry, "(connection has passed maxLifetime)", false /* not owner */)) {
                     //新增资源
                     addBagItem(connectionBag.getWaitingThreadCount());
                  }
               },
               lifetime, MILLISECONDS));
         }

         return poolEntry;
      }
      catch (ConnectionSetupException e) {
         if (poolState == POOL_NORMAL) { // we check POOL_NORMAL to avoid a flood of messages if shutdown() is running concurrently
            logger.error("{} - Error thrown while acquiring connection from data source", poolName, e.getCause());
            lastConnectionFailure.set(e);
         }
      }
      catch (Exception e) {
         if (poolState == POOL_NORMAL) { // we check POOL_NORMAL to avoid a flood of messages if shutdown() is running concurrently
            logger.debug("{} - Cannot acquire connection from data source", poolName, e);
         }
      }

      return null;
   }
```

1. 创建对象
2. 设置超时控制任务

### 定时控制数据库连接总数

> 默认周期30ms

```java
   private final class HouseKeeper implements Runnable
   {
      //记录上一周期时间点
      private volatile long previous = plusMillis(currentTime(), -housekeepingPeriodMs);

      @Override
      public void run()
      {
         try {
            //获取配置信息
            connectionTimeout = config.getConnectionTimeout();
            validationTimeout = config.getValidationTimeout();
            leakTaskFactory.updateLeakDetectionThreshold(config.getLeakDetectionThreshold());
            catalog = (config.getCatalog() != null && !config.getCatalog().equals(catalog)) ? config.getCatalog() : catalog;

            final long idleTimeout = config.getIdleTimeout();
            final long now = currentTime();

            //时间往前跳变时，软驱逐所有连接
            // Detect retrograde time, allowing +128ms as per NTP spec.
            if (plusMillis(now, 128) < plusMillis(previous, housekeepingPeriodMs)) {
               logger.warn("{} - Retrograde clock change detected (housekeeper delta={}), soft-evicting connections from pool.",
                           poolName, elapsedDisplayString(previous, now));
               previous = now;
               softEvictConnections();
               return;
            }
			//时间往后跳变时，不做处理
            else if (now > plusMillis(previous, (3 * housekeepingPeriodMs) / 2)) {
               // No point evicting for forward clock motion, this merely accelerates connection retirement anyway
               logger.warn("{} - Thread starvation or clock leap detected (housekeeper delta={}).", poolName, elapsedDisplayString(previous, now));
            }

            previous = now;

            String afterPrefix = "Pool ";
            if (idleTimeout > 0L && config.getMinimumIdle() < config.getMaximumPoolSize()) {
               logPoolState("Before cleanup ");
               afterPrefix = "After cleanup  ";
               //获取未被使用的对象列表，并计算需要删除的数量
               final List<PoolEntry> notInUse = connectionBag.values(STATE_NOT_IN_USE);
               int toRemove = notInUse.size() - config.getMinimumIdle();
               for (PoolEntry entry : notInUse) {
			      //空闲时间过长，关闭连接
                  if (toRemove > 0 && elapsedMillis(entry.lastAccessed, now) > idleTimeout && connectionBag.reserve(entry)) {
                     closeConnection(entry, "(connection has passed idleTimeout)");
                     toRemove--;
                  }
               }
            }

            logPoolState(afterPrefix);

            fillPool(); //重新填充池子，保证池子有足够的数据库连接
         }
         catch (Exception e) {
            logger.error("Unexpected exception in housekeeping task", e);
         }
      }
   }

```

关闭连接closeConnection:

```java
   void closeConnection(final PoolEntry poolEntry, final String closureReason)
   {
      //删除
      if (connectionBag.remove(poolEntry)) {
         final Connection connection = poolEntry.close();
         closeConnectionExecutor.execute(() -> {
            //异步关闭
            quietlyCloseConnection(connection, closureReason);
            if (poolState == POOL_NORMAL) {
               //填充新资源
               fillPool();
            }
         });
      }
   }
```

填充池子fillPool：

```java
   private synchronized void fillPool()
   {
      //待添加量=最小值（上限-已有连接数，空闲下限-已有空闲连接）- 已有任务数
      final int connectionsToAdd = Math.min(config.getMaximumPoolSize() - getTotalConnections(), config.getMinimumIdle() - getIdleConnections())
                                   - addConnectionQueueReadOnlyView.size();
      if (connectionsToAdd <= 0) logger.debug("{} - Fill pool skipped, pool is at sufficient level.", poolName);

      for (int i = 0; i < connectionsToAdd; i++) {
         //启动任务，分为中间任务或最后任务
         addConnectionExecutor.submit((i < connectionsToAdd - 1) ? poolEntryCreator : postFillPoolEntryCreator);
      }
   }
```

### Proxy数据库连接资源泄露检测任务ProxyLeakTask

com.zaxxer.hikari.pool.ProxyLeakTask

> 通过leakDetectionThreshold控制是否开启检测功能

```java
   @Override
   public void run()
   {
      isLeaked = true;

      final StackTraceElement[] stackTrace = exception.getStackTrace(); 
      final StackTraceElement[] trace = new StackTraceElement[stackTrace.length - 5];
      System.arraycopy(stackTrace, 5, trace, 0, trace.length);

      exception.setStackTrace(trace);
      LOGGER.warn("Connection leak detection triggered for {} on thread {}, stack trace follows", connectionName, threadName, exception);
   }

   void cancel()
   {
      scheduledFuture.cancel(false);
      if (isLeaked) {
         LOGGER.info("Previously reported leaked connection {} on thread {} was returned to the pool (unleaked)", connectionName, threadName);
      }
   }
```

> 如果没有及时cancel，则记录日志
