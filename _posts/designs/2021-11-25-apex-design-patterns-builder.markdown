---
layout: post
title: "Apex Design Patterns - 建造者模式"
subtitle: ""
date: 2021-11-25 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-design-patterns.png"
catalog: true
hidden: true
tags:
  - Salesforce
  - Apex
  - Design Patterns
---

## 建造者模式

建造者模式（Builder Pattern）是一种创建型设计模式，它允许你分步骤创建复杂对象。该模式通过将对象构造过程抽象化，使得同样的构建过程可以创建不同的表示。

### 定义

建造者模式是一种对象构建模式，它可以将复杂对象的建造过程抽象出来（抽象类别），使这个抽象过程的不同实现方法可以构造出不同表现（属性）的对象。

### 优势

1. 分离复杂对象的构建与表示
2. 提高代码的可读性和可维护性
3. 增加灵活性，便于扩展
4. 可以更好地控制构建过程
5. 支持对象的不可变性

## 问题示例

考虑以下类的实现：

```apex
public class Person {
    private final String firstName;
    private final String lastName;
    private final String email;
    private final String nickname;

    public Person(
        String firstName,
        String lastName,
        String email,
        String nickname
    ) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
        this.nickname = nickname;
    }
}
```

要实例化这个类，我们需要这样做：

```apex
Person pObj = new Person(
    'Peter',
    'Dong',
    'peter@example.com',
    'pdong'
);
```

### 问题分析

1. 构造函数需要传入多个参数，随着属性增多，可读性降低
2. 难以处理可选参数
3. 对象一旦创建，就无法修改属性（除非提供 setter 方法，但这会破坏对象的不可变性）

## 建造者模式解决方案

使用建造者模式优化后的代码：

```apex
public class Person {
    private final String firstName;
    private final String lastName;
    private final String email;
    private final String nickname;

    private Person(PersonBuilder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.email = builder.email;
        this.nickname = builder.nickname;
    }

    // Getter methods...

    public static class PersonBuilder {
        private String firstName;
        private String lastName;
        private String email;
        private String nickname;

        public PersonBuilder setFirstName(String firstName) {
            this.firstName = firstName;
            return this;
        }

        public PersonBuilder setLastName(String lastName) {
            this.lastName = lastName;
            return this;
        }

        public PersonBuilder setEmail(String email) {
            this.email = email;
            return this;
        }

        public PersonBuilder setNickname(String nickname) {
            this.nickname = nickname;
            return this;
        }

        public Person build() {
            return new Person(this);
        }
    }
}
```

现在，我们可以这样创建 Person 对象：

```apex
Person myPerson = new Person.PersonBuilder()
    .setFirstName('Peter')
    .setLastName('Dong')
    .setEmail('peter@example.com')
    .setNickname('pdong')
    .build();
```

### 优化要点

1. 创建一个静态内部类`PersonBuilder`
2. 每个 setter 方法返回 builder 本身，支持方法链
3. 提供一个`build()`方法来构造最终的`Person`对象
4. `Person`类的构造函数是私有的，只能通过 Builder 来创建

## 建造者模式的优势

1. **灵活性**：可以轻松添加或删除属性，而不影响客户端代码
2. **可读性**：方法链的方式使得对象创建过程更加清晰
3. **参数控制**：可以更好地处理可选参数
4. **不可变性**：可以创建不可变对象，同时提供灵活的构建过程
5. **验证**：可以在`build()`方法中添加参数验证逻辑

## 在 Salesforce 中的应用

在 Salesforce Apex 开发中，建造者模式可以在以下场景中发挥作用：

1. 构建复杂的 SOQL 查询
2. 创建测试数据
3. 构建复杂的 API 请求或响应对象
4. 配置复杂的可视化组件

## 注意事项

1. 对于简单对象，使用建造者模式可能会过度设计
2. 代码量会略微增加
3. 在并发环境中使用时需要注意线程安全性

## 结论

建造者模式是一种强大的设计模式，特别适合于创建复杂对象。在 Salesforce Apex 开发中，合理使用建造者模式可以提高代码的可读性、可维护性和灵活性。通过将对象的构建过程与其表示分离，我们可以更好地控制对象创建过程，并且可以轻松地创建不同表现的对象。

> 记住，设计模式是工具，不是规则。在应用建造者模式时，要根据具体情况和需求来决定，确保它真正为你的应用带来价值，而不是增加不必要的复杂性。