---
title: 设计模式：行为型Template method
date: 2020-6-21 16:33:54
tags: 设计模式
---

## 目的和场景

1. 父类固定流程整体框架，子类动态实现子步骤逻辑
2. 控制子类扩展点，仅开放特定的钩子函数方法
3. 重构时抽取相同代码，剥离不同的实现逻辑

## 现实案例

男生购物：

1. 买香烟
2. 支付宝付款
3. 回家

女生购物：

1. 买口红
2. 微信支付
3. 回家

> 流程一样，买的东西不一样，付款方式不一样

## 示例

```java
public abstract class ShoppingPeople {
    protected abstract String pickTarget();

    protected abstract void pay(String target);

    protected void goHome() {
        System.out.println("go home now");
    }

    public void go() {
        String target = pickTarget();
        System.out.println("pick target: " + target);
        pay(target);
        goHome();
    }
}

```

```java
public class ShoppingMan extends ShoppingPeople {

    @Override
    protected String pickTarget() {
        return "Cigarettes";
    }

    @Override
    protected void pay(String target) {
        System.out.println("Alipay");
    }
}
```

```java
public class ShoppingWoman extends ShoppingPeople {

    @Override
    protected String pickTarget() {
        return "Lipstick";
    }

    @Override
    protected void pay(String target) {
        System.out.println("WeChat");
    }
}
```

```java
public class Shopping {
    private ShoppingPeople people;

    public Shopping(ShoppingPeople people) {
        this.people = people;
    }

    public void go() {
        people.go();
    }

    public void changePeople(ShoppingPeople people) {
        this.people = people;
    }

    public static void main(String[] args) {
        ShoppingPeople man = new ShoppingMan();
        Shopping shopping = new Shopping(man);
        shopping.go();

        ShoppingPeople woman = new ShoppingWoman();
        shopping.changePeople(woman);
        shopping.go();
    }
}
```

## 源码参考

- java.util.Collections#sort()

## 参考

- https://java-design-patterns.com/patterns/template-method/
