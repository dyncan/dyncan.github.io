---
layout: post
title: "Apex Design Patterns - 单例模式"
subtitle: ""
date: 2021-11-26 12:00:00
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

### 单例模式(Singleton Design Pattern)

**什么是单例模式?**: 在单个事务上下文中，限制类的实例化仅为一个"单一"实例.

**为什么要用单例模式?**: 尽量减少对象实例化，以提高性能并减轻 Governor Limits 的影响.

让我们先来看个问题，然后尝试使用单例模式来解决。

问题:

```java
trigger AccountTrigger on Account (before insert, before update) {
    for(Account record : Trigger.new){
      AccountDemoRecordType rt = new AccountDemoRecordType();
      record.recordTypeId = rt.id;
    }
}

public class AccountDemoRecordType {
    public String id {get;private set;}
    public AccountDemoRecordType(){
        id = Account.sObjectType
              .getDescribe()
              .getRecordTypeInfosByName()
              .get('Demo')
              .getRecordTypeId();
    }
}
```

在上面的例子中，如果可以观察到随着批量插入/更新的记录越来越多, AccountDemoRecordType对象的实例每次都为每个记录创建一次, 这意味着当我们new一个对象时系统会帮我们申请内存地址，每一次去new的时候都会构建不同的地址, 造成资源浪费且效率不高, 而单例模式则保证了每次获取的实例化对为同一份.

解决方案:

```java
trigger AccountTrigger on Account (before insert, before update) {
    for(Account record : Trigger.new){
      AccountDemoRecordType rt = AccountDemoRecordType.getInstance();
      record.recordTypeId = rt.id;
    }
}

//惰性加载的机制: 也就是在使用的时候才去创建
public class AccountDemoRecordType {
  public String id {get;private set;}
  private AccountDemoRecordType(){
      id = Account.sObjectType
          .getDescribe()
          .getRecordTypeInfosByName()
          .get('Demo')
          .getRecordTypeId();
  }

  private static AccountDemoRecordType instance = null;

  public static AccountDemoRecordType getInstance() {
   if(instance == null) instance = new AccountDemoRecordType();
        return instance;
    }
}
```
