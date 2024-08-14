---
layout: post
title: "Apex 中的设计模式"
subtitle: ""
date: 2021-11-20 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-design-patterns.png"
catalog: true
tags:
  - Salesforce
  - Apex
  - Design Patterns
---

> [Design Pattern] is a solution to a problem in a context.

## 设计模式是什么？

我认为设计模式的本质是基于[OOP](https://zh.wikipedia.org/zh-cn/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1), 通过封装、继承、多态 (OOP 三大特性) 以及类的关联关系 + 组合关系来组成设计模式，同时在遵循单一职责原则、开闭原则、里氏替换原则、迪米特法则、依赖倒置原则、接口隔离原则及合成/聚合复用原则 (OOP 七大原则) 的前提下，被总结出来的经过反复实践并被多数人知晓且经过分类和设计的可重用的软件设计方式。

本文主要介绍在 Salesforce Apex 开发中常见的几种设计模式：

### 单例模式 (Singleton pattern)

单例模式是一种创建模式--它帮助你以一种特定的方式实例化对象。它允许一个对象只存在一个实例，但每个人都可以对该对象进行全局访问。单例模式可以帮助提高性能和减少内存占用。[Read More](https://dyncan.github.io/2021/11/26/apex-design-patterns-singleton/)

### 建造者模式 (Builder pattern)

建造者模式是一种对象创建型设计模式，其目的是将一个复杂的对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。这种模式允许一个用户使用不同的方式构建一个对象。[Read More](https://dyncan.github.io/2021/11/25/apex-design-patterns-builder/)

### 策略模式 (Strategy pattern)

策略模式允许在运行时选择和替换不同的算法或策略来解决特定问题。它通过定义一系列的算法和封装这些算法的类来实现这一目的，并使用一个上下文对象来管理算法的选择和执行。这样，当需要更改算法或策略时，只需要更改上下文对象使用的策略实现即可，而不需要修改使用该策略的代码。[Read More](https://dyncan.github.io/2021/11/28/apex-design-patterns-strategy/)

### 外观模式 (Facade pattern)

外观模式提供了一个简单的接口来访问一组复杂的子系统。外观模式为子系统中的一组接口提供了一个统一的入口，这样，客户端就不必直接访问子系统中的对象，而是通过外观类访问这些对象。外观模式的目的是简化接口，并将客户端与子系统的实现细节隔离开来。[Read More](https://dyncan.github.io/2021/12/02/apex-design-patterns-facade/)