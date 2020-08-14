---
title: 分布式：一致性散列方法
date: 2020-8-2 16:33:54
tags: 分布式
---

## 普通散列

常见场景：

2个实例或多个实例作为后端存储或缓存，数据通过散列方法分布在不同的实例上

常用方法：

散列后取模，例如`idx = Zlib.crc32(key) % servers.size`

通过对key进行散列后，对实例数进行取模，获取到对应的实例索引

常见问题：

当扩容或缩容实例时，所有数据要进行重新散列重排，对资源消耗和性能影响非常大

## 一致性散列

### 原理

![alt](http://michaelnielsen.org/blog/wp-content/uploads/2009/06/consistent_hashing_3.PNG)

1. 取模确定数据范围为环形状态
2. 确定散列范围，比如SHA-1，总位数有2^160
3. 对实例进行散列，使其散列值均匀的分布在环上
4. 按照一定规则，例如顺时针，散列值落在实例0和实例1范围上的数据，存储在实例1上

### 扩容

> 缩容类型，动作相反

![alt](http://michaelnielsen.org/blog/wp-content/uploads/2009/06/consistent_hashing_4.PNG)

1. 新增实例3，确定其散列值
2. 需要将原先在实例1上散列值范围在实例0和实例3上的数据迁移至实例3

优点：数据影响范围较小

缺点：实例负载分布不均

### 优化

![alt](http://paperplanes-assets.s3.amazonaws.com/consistent-hashing.png)

1. 优化实例的散列方法
2. 由散列值的点改为散列范围点，保证各个实例负责的范围点，比较均匀分布在环上

优点：

1. 减少实例负载热点
2. 快速扩容或缩容
3. 高效支持副本功能
4. 提高服务可扩展性和可用性

## 参考

- http://michaelnielsen.org/blog/consistent-hashing/
- https://www.paperplanes.de/2011/12/9/the-magic-of-consistent-hashing.html
