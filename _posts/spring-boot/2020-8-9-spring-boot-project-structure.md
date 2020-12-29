---
title: SpringBoot：Web应用项目结构
date: 2020-8-9 16:02:06
tags: SpringBoot
---

# SpringBoot：Web应用项目结构

## 分层

1. 3rd，存放调用第三方系统的Adapter
2. common，存放公共方法或Bean
3. controller，存放对外提交API
4. repository，存放和DB交互的类，采用Mybatis框架
5. service，存放具体功能模块的实现

## 依赖

**模块**

controller -> service -> (common, 3rd, repository)

**开源包**

根pom.xml声明所有软件及版本号，子pom.xml声明父子关系及依赖软件的坐标（无版本号）

## 示例

```
D:.
├─3rd
|   │  pom.xml
├─common
|   │  pom.xml
|-controller
|   │  pom.xml
|-repository
|   │  pom.xml
└─service
    │  pom.xml
```
