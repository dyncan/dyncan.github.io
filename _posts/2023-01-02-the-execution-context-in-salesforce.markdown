---
layout: post
title: "「译」Salesforce 中的执行上下文"
subtitle: ""
date: 2023-01-02 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-execution-context-salseforce.jpeg"
catalog: true
tags:
  - Apex
  - Execution Context
  - Salesforce
---

本篇文章的译文内容节选自[「Advanced Apex Programming - Fifth Edition」](https://advancedapex.com/)第一章的第一部分 - `Thinking in Apex`, 该书的作者是 [Dan Appleman](https://twitter.com/danappleman). 个人感觉这本书写的不错, 但是国内目前没有发现有中文版本, 所以就尝试着翻译这本书, 该书共有三大章节, 13个小节, 只在自己空闲的时间翻译一点, 目前第一章的进度大概在 70% 左右, 不过由于书籍版权的问题, 翻译的所有内容暂时不会公开.

![img](/img/in-post/post-bg-advanced-apex-programming.png)

**翻译这本书的目的:** 一来可以作为一个知识学习的渠道, 二来也可以提高自己的英文读写能力.

下面是本书第一章 `The Execution Context` 的译文:

---

执行环境上下文是 Apex 编程中的关键定义概念之一. 它影响着 Salesforce 平台上软件开发的每一个方面.

一个执行环境上下文有两个特点:

* 它定义了静态变量的范围和生命周期.
* 它为那些在执行上下文之间被重置的 `governor limits` 定义了上下文.

我将在后面更深入地讨论 `静态变量` 和 `limits`. 现在, 我们需要记住的关键事实(我向你保证, 一旦你开始在 Apex 中工作, 你将永远不会忘记它们)是:

* 静态变量在执行上下文期间一直保留, 且对于一个执行上下文是唯一的.
* 许多(但不是全部) limits 在执行上下文之间重置. 例如, 如果governor limits限制您在执行上下文中只能进行 100 次数据库查询, 则每个执行上下文可以有 100 次数据库查询. 不同类型的执行上下文可能有不同的限制.
* 你能够知道一个执行上下文何时开始. 一般你不能知道它何时结束.

## Running Apex Code

当发生一系列可能的外部事件或操作时, 一个执行上下文就会开始, 这些事件或操作具有运行 Apex 代码的能力. 这些事件包括:

* **一个数据库触发器(Database trigger):** 触发器可以在许多标准 Salesforce 对象和所有自定义对象的插入, 更新, 删除或取消删除时发生.
* **Future Call(异步调用):** Future 调用可以从 Apex 代码中请求. Future可以有更高的限制来运行代码.
* **Queueable Apex:** 与future call类似,这是另一种异步运行代码的机制.
* **Scheduled Apex:** 您可以实现一个可以被系统按计划调用的 Apex 类.
* **Batch Apex:** 你可以实现一个专门用于处理大量数据记录的类.
* **Platform Events:** Apex可以触发Platform Events - Salesforce的企业消息队列.
* **Change Data Capture(CDC):** 可以将其看作是建立在Platform Events服务上的异步数据库触发器.
* **Web service:** 你可以实现一个类, 可以通过 SOAP 或 REST 从外部网站或从网页上的 JavaScript 访问.
* **VisualForce and Lightning components:** 您的 VisualForce 页面和 Lightning组件可以在控制器中执行Apex代码或设置页面属性或执行方法.
* **Lightning Processes and Flows:** 这些是声明性的编程工具, 可以调用Apex类中的方法.
* **Global Apex:** 你可以公开一个全局方法, 也可以从其他 Apex 代码中调用.
* **Inbound Email Service:** 当邮件到达指定的email address时, 可以调用Apex.
* **Anonymous Apex:** Apex 代码可以从开发者控制台, SFDX或通过外部网络服务调用并动态地编译和执行.

当 Apex 的代码由于任何上述这些事件或操作而开始执行时, 那么它就会在一个执行上下文中运行. 你的代码总是可以通过调用 [Request.getQuiddity](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_class_System_Request.htm#apex_System_Request_getQuiddity) 方法来确定什么类型的事件或操作启动了当前的执行环境.

所有由于原始请求而同步运行的后续 Apex 代码将继续在同一执行上下文中运行. 作为异步请求的结果, 异步操作将在其自身的执行上下文中运行.

考虑在 Lead 对象更新时触发的简单情况. 在一个只有一个触发器的全新Org的情况下, 如图 1-1 所示, 上下文在您的触发器代码开始运行时开始, 在您的代码退出时结束:

<p align="center">
    <img src="/img/in-post/trigger-execution-context_1-1.png">
</p>

如果您在插入 Lead 时有多个触发器怎么办? 这并不罕见, Org中的代码会随着时间推移而发展, 一个开发人员在同一事件上建立新触发器很常见. 而不是冒险修改(和破坏)另一个开发人员编写的触发器代码. 现在您可以得到如图 1-2 所示的场景:

<p align="center">
    <img src="/img/in-post/trigger-execution-context_1-2.png">
</p>

正如你所看到的, 两个触发器都在同一个执行上下文中运行. 所以它们共享同一组限制(只要它们在同一个应用程序中--后面会详细介绍), 以及同一组静态变量.
这似乎很简单. 但是, 如果在 Lead 上有一个做字段更新的工作流呢? 现在你可能会出现图 1-3 所示的情况:

<p align="center">
    <img src="/img/in-post/trigger-execution-context_1-3.png">
</p>

正如你所看到的, 字段更新工作流不仅在同一执行上下文中运行, 它还能使触发器在同一上下文中再次执行.

这就带来了一个有趣的问题. 你怎么能知道你的触发器是第一次执行, 还是因为工作流或其他触发器而再次执行同一个更新?

这就是静态变量发挥作用的地方. 请记住, 它们的生命周期和范围是由执行环境定义的. 所以, 如果你有一个静态变量定义在类 "myclass "中, 像这样:

```java

public Static Boolean firstcall = false;

```

你可以在你的触发器中使用下面的设计模式来确定这是这个执行上下文的第一次调用还是后续调用.

```java

if(!myclass.firstcall)
{
    // First call into trigger
    myclass.firstcall = true;
}
else {
    // Subsequent call into trigger
}

```

这种设计模式最终被证明是非常重要的, 正如你稍后会看到的.

让我们考虑一下你们刚才看到的情况的后果:

* 你可以在一个事件上有多个触发器, 但不能控制它们的执行顺序.
* 限制是在执行上下文中共享的; 因此, 你可能会与其他代码共享限制, 而这些代码你无法控制, 而且可能是在你的代码建立和测试后添加的.
* 工作流, flow 和 Lightning process等可以由非程序员创建, 它们会以不可预测的方式影响你的 Apex 代码的顺序和执行, 甚至导致你的代码在同一对象和同一执行环境中再次执行.

在这一点上, 如果你从其他语言转到 Apex, 你可能会感到一定程度的震惊. 在其他平台上, 你作为开发人员对你的应用程序有很大的控制权. 用户输入什么可能是不可预测的, 也必须要考虑到, 但你知道如何处理. 用户不会修改应用程序的底层行为或添加与你的应用程序交互的代码. 即使你有中间件或 API, 也希望使用它的人遵守你的规范, 指南等.

但 Salesforce 的设计是为了尽量减少对定制软件的需求. 事实上, Salesforce是日益流行的`低代码`应用开发趋势的先驱. 因此, 用户做的一些事情都会影响你写的代码. 因为执行上下文是共享的, 所以其他开发者可以做一些事情来影响你写的代码.

我所说的影响, 是指中断.

但是不要让这个想法吓到你. 因为你会很快适应这样的想法: 你精心打造的代码可能会被一个粗心的初级系统管理员写的验证规则破坏. 这只是领域的一部分——也是 Apex 编码充满乐趣的挑战的一部分.

我在这里的意思不是要吓唬你, 而是要强调我之前的观点. Apex 可能看起来像 Java, 但由于基本的平台差异, 它需要一套不同的设计模式和不同的方法来架构应用程序.