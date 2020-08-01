---
title: 设计入门：组件和连接
date: 2020-7-12 16:01:01
tags: 系统设计
---

## 组件和连接

![alt](https://raw.githubusercontent.com/xied5531-forked/system-design-primer/master/images/jrUBAF7.png)

1. Client通过DNS解析出目标地址，提供首层负载均衡能力
2. Client通过CDN就近获取缓存资源，降低后端服务压力
3. LoadBalancer通过实例负载均衡实现用户请求分担，降低后端Web Server压力
4. Web Server提供对外API服务，实现接口权限、资源等管理
5. 操作类型区分Read API、Write API、Search API，为读写搜索操作分别进行精细化优化
6. Memory Cache提供Service和Data Store之间缓存，降低数据访问压力
7. SQL和Object分别提供关系化SQL和非关系化NOSQL数据存储服务
8. DB通过读写分离、分库分表等动作提升数据库性能

> 通用系统设计组件模型，根据业务实际应用场景进行局部调整优化

## 参考

- https://github.com/xied5531-forked/system-design-primer/blob/master/README-zh-Hans.md
