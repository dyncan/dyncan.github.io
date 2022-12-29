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

> [Design Pattern] is a solution to a problem in a context.

### 建造者模式 (Builder pattern)

**什么是建造者模式?**: 是一种对象构建模式, 它可以将复杂对象的建造过程抽象出来（抽象类别）,使这个抽象过程的不同实现方法可以构造出不同表现（属性）的对象.

**为什么要用建造者模式?**: 首先Builder Pattern是一个创建型模式, 它可以用于将一个复杂对象的构建与它的表示分离, 这样做的好处是可以使得创建复杂对象的过程更加灵活和可控, 使代码更加可读、可维护，也更容易扩展.

让我们先来看个问题，然后尝试使用建造者模式来解决。

问题:

```java
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

那么我们如何实例化上边的Class呢?

```java
Person pObj = new Person(
 'Peter',
 'Dong',
 'peter@example.com',
 'pdong'
);
```

我们发现创建实例化的过程构造函数需要传入多个参数, 如果随着对象的属性的增多可能使之后的维护人员更难理解, 可读性非常差, 另外如果一个属性是可选的话上述方法还是需要传入值的.

解决方案:

```java
public class PersonBuilder {
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
   // ...
   
   public Person build() {
     return new Person(
       this.firstName, this.lastName, this.email, this.nickname
     );
   }
 }
```

实例化:

```java
Person myPerson = new PersonBuilder()
                          .setFirstName('Peter')
                          .setLastName('Dong')
                          .setEmail('peter@example.com')
                          .setNickName('pdong')
                          .build();
```
