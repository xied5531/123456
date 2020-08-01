---
title: 设计模式：创建型Builder
date: 2020-7-19 16:36:02
tags: 设计模式
mermaid: true
---

## 目的和场景

1. 创建复杂对象
2. 创建和组装过程独立
3. 通过组装可以创建不同属性的对象

## 现实案例

游戏角色创建：

1. 创建游戏角色
2. 选择性别
3. 选择职业
4. 选择发型
5. 选择服装
6. 选择。。。
7. 创建角色

## 示例

```java
public class Hero {
    private final String profession;

    private final String name;

    private final String hairColor;

    private final String weapon;

    private Hero(Builder builder) {
        this.profession = builder.profession;
        this.name = builder.name;
        this.hairColor = builder.hairColor;
        this.weapon = builder.weapon;
    }

    @Override
    public String toString() {
        return "Hero{" +
            "profession='" + profession + '\'' +
            ", name='" + name + '\'' +
            ", hairColor='" + hairColor + '\'' +
            ", weapon='" + weapon + '\'' +
            '}';
    }

    public static class Builder {
        private final String profession;

        private final String name;

        private String hairColor;

        private String weapon;

        public Builder(String profession, String name) {
            if (profession == null || name == null) {
                throw new IllegalArgumentException("profession and name can not be null");
            }
            this.profession = profession;
            this.name = name;
        }

        public Builder withHairColor(String hairColor) {
            this.hairColor = hairColor;
            return this;
        }

        public Builder withWeapon(String weapon) {
            this.weapon = weapon;
            return this;
        }

        public Hero build() {
            return new Hero(this);
        }
    }

    public static void main(String[] args) {
        Hero hero = new Hero.Builder("teacher", "bob").withHairColor("black").withWeapon("knife").build();
        System.out.println(hero);
    }
}
```

> Builder构造函数提供默认最小配置，同样可以设置默认值，其他方法提供附加配置，

更加复杂场景：

1. 定义抽象Builder，提供默认最小化配置
2. 定义具体Builder，实现具体附加配置方法，生产具体不同属性的对象

## 源码参考

- JDK：java.lang.StringBuffer，append方法
- JDK：java.lang.StringBuilder，append方法


## 参考

- https://java-design-patterns.com/patterns/builder/
