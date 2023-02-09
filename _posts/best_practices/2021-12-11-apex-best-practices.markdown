---
layout: post
title: "Apex 最佳实践"
subtitle: ""
date: 2021-12-11 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-apex-best-practices.png"
catalog: true
tags:
  - Salesforce
  - Apex
  - Best Practices
---

> A good Salesforce engineer knows how to use Apex. A great Salesforce engineer knows when to not use Apex.

当我们在 Salesforce 平台构建解决方案的时候, 首先考虑使用平台的内置功能, 比如Validation rules,  flows等, 这些功能可以提供更好的性能,  而且通常更容易维护.  然而在有些情况下使用代码解决问题反而是性能更好的, 特别是遇到比较复杂的嵌套循环和外部系统集成等场景, Apex 倾向于提供更多的灵活性, 而且在大数据量的记录中表现得更好. 所以一个好的 Salesforce 工程师会有意识地在每种情况下权衡代码与配置的优势, 从而找出最佳方案. 接下来会介绍几种在使用 Apex 的时候, 遵循的几种最佳实践:

接下来会介绍几种在使用 Apex 的时候, 遵循的几种最佳实践:

### 1. 批量处理你的代码

代码的批量化是指: 使你的代码能够一次有效的处理多条记录的过程, 主要适用于 Apex Trigger, 但是个人认为在常规的 Apex 代码中进行 DML 操作时,  批量化代码也尤为重要, 这样你的代码就能正确处理不同情况下的所有记录, 以提高你的代码性能.

下面的触发器假设只有一条记录触发 Trigger. 当在同一个事务中插入多条记录时, 这个触发器对其他所有的记录会不起作用:

```java
trigger MyTriggerNotBulk on Opportunity(before insert) {
    Opportunity opp = Trigger.new[0];
    opp.Description = 'New description';
}
```
下面这个例子是 MyTrigger 的一个更新版本. 是为处理触发器中的批量记录而修改的代码, 这意味着所有记录将在一次触发器调用中被处理:

```java
trigger MyTriggerBulk on Opportunity(before insert) {
    for(Opportunity opp : Trigger.new) {
        opp.Description = 'New description';
    }
}
```

另外也可以通过使用 Map 来降低代码复杂度, 减少代码运行时间. 下面的一个例子是在处理过程中使用 Map 来更有效地访问记录的数据:

```java
Set<Id> accountIds = new Set<Id>();

for (Opportunity opp :Trigger.new) {
  accountIds.add(opp.AccountId); 
}

Map<Id, Account> accountById = new Map<Id, Account>(
  [ SELECT Id,Name FROM Account WHERE Id IN :accountIds]
);

for (Opportunity opp :Trigger.new) {
  opp.Account_Name__c = accountById.get(opp.AccountId).Name; 
}
```

### 2. 避免循环中使用 DML/SOQL 查询

SOQL 查询和 DML 操作是在 Apex 中最昂贵的操作之一(会消耗 CPU 时间), 两者也有严格的执行限制, 因此将这些操作放在 for 循环中会是一种灾难. 因为我们可以在不知不觉中迅速达到 Salesforce Limits, 特别是在涉及到 Apex Trigger 的时候.

对于 DML 语句, 我们可以将这些语句转移到循环外, 相反, 在我们的循环中, 我们可以将我们希望执行这些操作的记录添加到一个 List/Map 中, 然后对我们的列表执行 DML 语句. 对于几乎所有情况, 这是最安全和最好的方法:

```java
List<Account> accountsToUpdate = new List<Account>(); 

for (Opportunity opp : Trigger.new){
  if (opp.StageName == 'Closed Won'){
    Account account = new Account(
        Id = opp.AcCountId,
        Has_Closed_Won_Opp__c = true
    );

    accountsToUpdate.add(account)
  }
}

update accountsToUpdate;
```

### 3. 避免硬编码 ID

在开发阶段, 硬编码 ID 可能没有任何问题, 但是一旦将代码部署到生产环境中, 这些 ID 的引用就会失效. 比如记录类型ID, 很容易被硬编码. 我们如果希望利用记录类型来查询数据, 我们可以通过 Name/Develope Name 来引用,  这样可以在不同环境保持一致:

```java
  public static final Id RECORD_TYPE_ID = Schema.SObjectType
                          .Account
                          .getRecordTypeInfosByName()
                          .get('Business Account')
                          .getRecordTypeId();
```

如果我们希望使用的 ID 与一个特定的记录有关, 我们可以把这个 ID 存储到自定义元数据中, 在执行查询时检索这个值, 这些设置允许我们在不同的 ORG 中改变这个值, 或者随着需求的改变而改变.

### 4. 明确声明共享模式

当我们开始编写一个全新的类时, 我们应该做的第一件事就是声明我们的共享模式, 明确地声明我们的共享模式, 可以让我们向未来在我们的代码上工作的其他同事(甚至可能是你自己!)展示我们的意图. 这可以让他们更容易理解代码中发生的事情. 关于 Sharing Mode, [Stackexchange](https://salesforce.stackexchange.com/questions/264509/inherited-sharing-vs-no-sharing-declaration) 上的一个回答讲得不错.

### 5. 模块化你的代码

想象一下场景: 你已经写了一个方法来帮助建立动态 SOQL 查询. 另一个需求出现了, 你发现可以使用这个方法, 所以你把代码复制到你的新类中, 而且执行起来没有问题, 几天后, 在你的方法中发现了一个相当严重的错误--你及时地修复了它.但是现在你需要去把这个问题在你加入的另一个类中也要修复掉. 长此以往, 代码会很快就变得不可维护, 因为你需要更新它被复制到的每一个地方, 而且每次都会因为人为错误而带来更大的错误风险.

所以, 我们应该做的是把这些可重用的代码放在他们自己的独立的类中, 并在你需要该功能的地方调用这些类和方法. 这可以大大降低你的代码的复杂性, 而且当你的模块中发现错误时, 只需要修复一次, 无论在哪里使用这些代码, 它都会被修复, 也不会出错.

### 6. 约定一个命名规则

[命名规则](https://trailhead.salesforce.com/content/learn/modules/success-cloud-coding-conventions/choose-naming-conventions-sc)往往是任何开发团队中的一个热门话题. 如果每个人都遵守规范, 好处是显而易见的, 因为它们使你团队中的其他人更加容易理解代码中发生的事情. 在做代码 Review的时候效率也更高, 也能更清晰的了解代码的结构和意义, 你可以把更多的时间花在其他更重要的事情上.

### 7. 避免将 JSON 返回给 Lightning 组件

当我们为Lightning组件编写 `@AuraEnable` 方法时, 我们经常需要返回更复杂的数据结构, 如记录,自定义数据类型, 或这些类型的列表.

一个简单的方法是将这些对象序列化为JSON, 然后在我们组件的 JavaScript 中反序列化:

```java
  @AuraEnabLed( cacheable=true )
  public static String getAccounts() {
    return JSON.serialize([SELECT Id FROM Account]);
  } 
```

然而,这种方法实际上是一种`anti-pattern`, 会导致我们的组件出现一些糟糕的性能. 相反, 我们应该直接返回这些对象, 而让平台为我们处理其余的事情.

```java
  @AuraEnabLed( cacheable=true )
  public static String getAccounts() {
    return [SELECT Id FROM Account];
  }
```

将我们返回的数据转换为 JSON 会导致我们消耗大量的堆内存, 并耗费更多的 CPU 时间将这些对象转换为一个长字符串. 如果有一组相当复杂的处理过程, 或者也许我们有大量的记录需要返回, 我们很快就会遇到 Governor Limits.

当我们直接返回结果时, 序列化为 JSON 是由平台处理的, 这些操作不会计入我们的 Governor Limits. 结果也会自动转换为对象, 让我们的 Lightning 组件使用.

## 总结

无论从短期还是长期来看, 最佳实践都应该成为每个开发者的重要组成部分. 知道为什么和什么可以帮助我们成长为更好的工程师, 使我们能够对我们的选择可能产生的影响做出明智的, 更聪明的决定.
