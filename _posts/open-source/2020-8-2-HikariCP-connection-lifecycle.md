---
title: HikariCP：数据库连接生命周期
date: 2020-8-2 14:40:52
tags: [HikariCP]
---

## 核心组件

1. HikariDataSource，入口管理程序，负责getConnection()
2. HikariPool，连接资源池管理模块，负责资源池整体状态的管理
3. ConcurrentBag，资源池存储模块，负责资源的增加、删除、借用、归还
4. PoolEntry，资源池对象，对ProxyConnection的封装，记录状态、时间等
5. ProxyFactory，代理数据库连接工厂，负责管理数据库连接代理
6. ProxyConnection，代理数据库连接对象，对原始Connection的封装，并代理close()

## 生命周期

```mermaid
sequenceDiagram
    participant DS as HikariDataSource
    participant Pool as HikariPool
    participant Bag as ConcurrentBag
    participant Entry as PoolEntry
    participant Factory as ProxyFactory
    participant Connection as ProxyConnection

    opt getConnection()
        DS->>Pool: getConnection()
        Pool->>Bag: borrow()
        Bag->Entry: 
        Entry->>Factory: getProxyConnection()
        Factory->Connection: 
        
        Connection-->Factory: 
        Factory-->>Entry: 
        Entry-->>Pool: 
        Pool-->>DS: ProxyConnection
    end
    
    opt close()
        Connection->>Connection: close()
        Connection->>Entry: recycle()
        Entry->>Pool: recycle()
        Pool->>Bag: require()
    end
```

- getConnection()，获取数据库连接
- close()，关闭数据库连接
