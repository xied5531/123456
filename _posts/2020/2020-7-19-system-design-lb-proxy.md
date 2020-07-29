---
title: 系统设计：网络LB和Proxy
date: 2020-7-19 16:36:02
tags: 系统设计
---

> LB和Proxy作用类似，Proxy主流做法大部分都提供部分负载均衡能力

## LB作用

1. 服务发现：有哪些后端实例提供服务，以及怎么找到他们
2. 健康检查：哪些后端实例是可用的
3. 负载均衡：用什么算法对后端实例做请求分发

> 根据LB作用的位置，对应OSI模型，分为L4和L7

## 4层LB

![L4](https://miro.medium.com/max/700/1*1PjTpM3hLnm3iEAd4-_AaQ.png)

L4：通常基于TCP/UDP协议层，用于处理Connection和Session

L4：为每个Session通过中间人LB建立输入的session和输出的session

问题：当不同的客户端请求速率不一样时，L4的策略会严重影响不同客户端请求的处理效率

## 7层LB

![L7](https://miro.medium.com/max/700/1*zsaxjSziEm1Tipr6kxs5Zg.png)

L7：通常基于HTTP应用层，用于处理application应用层的连接

L7：为每个HTTP请求通过中间人LB建立输入连接和后端的多个输出连接

为不同的客户端请求速率，提供统一的负载均衡策略

## 常用LB模型

### Global Server Load Balancing (GSLB)

![gslb](https://www.f5.com/content/dam/f5-com/page-assets-en/home-en/company/blog/2018/global_server_load_balancing.jpg)

> 通常基于DNS做站点级LB

1. 通过GSLB获取可用的站点信息
2. 返回可用站点的LB信息

### Plain Old Load Balancing (POLB)

![polb](https://www.f5.com/content/dam/f5-com/page-assets-en/home-en/company/blog/2018/plain-old_load_balancing.jpg)

> 基于TCP连接的LB

通过实例克隆，选择可用的实例和客户端之间建立连接

### Persistence

![pblb](https://www.f5.com/content/dam/f5-com/page-assets-en/home-en/company/blog/2018/persistence-based_load_balancing.jpg)

> 基于Session的LB

通过记录请求的Cookie信息，关联cookie和实例信息

### Host-Based Load Balancing

![hblb](https://www.f5.com/content/dam/f5-com/page-assets-en/home-en/company/blog/2018/host-based_load_balancing.jpg)

> 基于虚拟主机的LB

通过虚拟主机服务，对不同的请求地址提供不同虚拟主机，实现负载均衡

### Route and Return

![hblb](https://www.f5.com/content/dam/f5-com/page-assets-en/home-en/company/blog/2018/http-based_load_balancing.jpg)

> 基于请求路径的7层LB

通过匹配请求路径信息，提供不同的资源池，实现请求负载均衡

## 参考

- https://blog.envoyproxy.io/introduction-to-modern-network-load-balancing-and-proxying-a57f6ff80236
- https://www.f5.com/company/blog/top-five-scalability-patterns
