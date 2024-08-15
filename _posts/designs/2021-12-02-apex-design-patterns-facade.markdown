---
layout: post
title: "Apex Design Patterns - 外观模式"
subtitle: ""
date: 2021-12-02 12:00:00
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

### 外观模式 (Facade pattern)

**什么是外观模式？**: 这是一种结构设计模式，它隐藏了系统的复杂性，并向客户提供了简单的接口。这种模式涉及一个名为 Facade 的类，该类提供客户端所需的简化方法，并将其调用委托给现有类的方法

策略模式使得算法可以在不影响到客户端的情况下发生变化。其主要目的是通过定义相似的算法，替换 if else 语句写法，并且可以随时相互替换

**为什么要用外观模式？**: 在系统中可能有许多复杂的类。由于它们之间有许多依赖关系，因此在许多地方调用它们的方法可能是繁琐且重复的过程。因此，我们创建了一个名为 Façade 的简单类，它在其实现中利用来自这些复杂类的方法，但向客户提供一个简单的接口。

问题：假设你要画圆，方和矩形，你有三个不同的类来对应这些画法。

解决方案：我们将创建一个 facade 类，通过隐藏底层的复杂性，提供一个简单的接口来绘制所有这三种形状。

```java
public interface Shape {
   void draw();
}
```

```java
public class Rectangle implements Shape {
 
   public void draw() {
      System.debug('Rectangle::draw()');
   }
}
```

```java
public class Square implements Shape {
   public void draw() {
      System.debug('Square::draw()');
   }
}
```

```java
public class Circle implements Shape {
 
   public void draw() {
      System.debug('Circle::draw()');
   }
}
```

```java
public class ShapeFacade {
   private Shape circle;
   private Shape rectangle;
   private Shape square;
 
   public ShapeFacade() {
      circle = new Circle();
      rectangle = new Rectangle();
      square = new Square();
   }
 
   public void drawCircle(){
      circle.draw();
   }
   public void drawRectangle(){
      rectangle.draw();
   }
   public void drawSquare(){
      square.draw();
   }
}
```

```java
ShapeFacade shapeMaker = new ShapeFacade();
shapeMaker.drawCircle();
shapeMaker.drawRectangle();
shapeMaker.drawSquare();
```

在 Salesforce 中，你可以使用 facade 设计模式，通过为客户提供一个简单的接口来获得响应，从而隐藏外部调用的潜在复杂性。



