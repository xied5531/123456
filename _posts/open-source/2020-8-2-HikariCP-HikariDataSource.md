---
title: HikariCP：入口管理程序HikariDataSource
date: 2020-8-2 15:40:52
tags: [HikariCP]
---

## 属性

外置属性：

`HikariConfig`, 统一配置管理，部分支持动态配置

内置属性：

```java
   private final AtomicBoolean isShutdown = new AtomicBoolean();	//当前状态
   private final HikariPool fastPathPool;	//资源池引用
   private volatile HikariPool pool;		//资源池引用
```

## 初始化

```java
   public HikariDataSource()
   {
      super();
      fastPathPool = null;
   }
```

> 默认配置，先不进行实际初始化，待首次请求过来时进行初始化

```java
   public HikariDataSource(HikariConfig configuration)
   {
      configuration.validate();	//校验配置
      configuration.copyStateTo(this);	//复制配置信息，避免configuration变化后有影响

      LOGGER.info("{} - Starting...", configuration.getPoolName());
      pool = fastPathPool = new HikariPool(this);
      LOGGER.info("{} - Start completed.", configuration.getPoolName());

      this.seal();
   }
```

> 指定配置，执行进行初始化，并标记状态

## getConnection()

```java
   public Connection getConnection() throws SQLException
   {
      //判断状态
      if (isClosed()) {
         throw new SQLException("HikariDataSource " + this + " has been closed.");
      }

      //判断是否已经初始化过
      if (fastPathPool != null) {
         return fastPathPool.getConnection();
      }

      // See http://en.wikipedia.org/wiki/Double-checked_locking#Usage_in_Java
	  // 双重锁校验，控制并发初始化HikariPool
      HikariPool result = pool;
      if (result == null) {
         synchronized (this) {
            result = pool;
            if (result == null) {
               validate();
               LOGGER.info("{} - Starting...", getPoolName());
               try {
                  pool = result = new HikariPool(this);
                  this.seal();
               }
               catch (PoolInitializationException pie) {
                  if (pie.getCause() instanceof SQLException) {
                     throw (SQLException) pie.getCause();
                  }
                  else {
                     throw pie;
                  }
               }
               LOGGER.info("{} - Start completed.", getPoolName());
            }
         }
      }

      return result.getConnection();
   }
```

> 初始化时对持有的HikariPool进行初始化，并通过HikariPool进行getConnection()
