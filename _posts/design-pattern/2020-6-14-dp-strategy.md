---
title: 设计模式：行为型Strategy
date: 2020-6-14 16:33:54
tags: 设计模式
---

## 目的和场景

1. 多个相关的类，只有行为不同，可以通过配置快速改变行为
2. 针对不同的场景，实现特定的优化算法，可以根据运行环境快速改变算法
3. 隐藏不同算法实现细节

## 现实案例

购物：

1. 选目标
2. 付款
  - 支付宝
  - 微信

> 买的东西一样，付款方式不一样

## 示例

```java
public interface Pay {
    void pay(String target);
}

```

```java
public class Alipay implements Pay {
    @Override
    public void pay(String target) {
        System.out.println("Alipay:" + target);
    }
}
```

```java
public class WeChatpay implements Pay {
    @Override
    public void pay(String target) {
        System.out.println("WeChatpay:" + target);
    }
}
```

```java
public class Shopping {
    private Pay pay;

    public Shopping(Pay pay) {
        this.pay = pay;
    }

    public void go() {
        String target = "pencil";

        System.out.println("target:" + target);
        pay.pay(target);
    }

    public void changePay(Pay pay) {
        this.pay = pay;
    }

    public static void main(String[] args) {

        Pay pay = new Alipay();
        Shopping shopping = new Shopping(pay);
        shopping.go();

        pay = new WeChatpay();
        shopping.changePay(pay);
        shopping.go();
    }
}
```

## 区别联系

Template method：

1. 注重流程固定，各阶段行为不同
2. 采用抽象类

Strategy：

1. 注重算法不同，快速替换
2. 采用接口

## 源码参考

- java.util.Comparator#compare()

## 参考

- https://java-design-patterns.com/patterns/strategy/
