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

## 单例模式

单例模式（Singleton Design Pattern）是一种常用的软件设计模式，它确保一个类只有一个实例，并提供一个全局访问点。

### 定义
在单个事务上下文中，限制类的实例化仅为一个"单一"实例。

### 优势
1. 减少对象实例化，提高性能
2. 降低内存使用，减轻 Salesforce Governor Limits 的影响
3. 确保全局状态的一致性

## 问题示例

考虑以下触发器和类的实现：

```apex
trigger AccountTrigger on Account (before insert, before update) {
    for(Account record : Trigger.new){
        AccountDemoRecordType rt = new AccountDemoRecordType();
        record.recordTypeId = rt.id;
    }
}

public class AccountDemoRecordType {
    public String id {get; private set;}
    
    public AccountDemoRecordType(){
        id = Account.sObjectType
            .getDescribe()
            .getRecordTypeInfosByName()
            .get('Demo')
            .getRecordTypeId();
    }
}
```

### 问题分析
1. 每次处理记录时都创建`AccountDemoRecordType`的新实例
2. 随着批量操作记录数量增加，不必要的对象创建也会增加
3. 造成资源浪费和性能下降

## 单例模式解决方案

使用单例模式优化后的代码：

```apex
trigger AccountTrigger on Account (before insert, before update) {
    for(Account record : Trigger.new){
        AccountDemoRecordType rt = AccountDemoRecordType.getInstance();
        record.recordTypeId = rt.id;
    }
}

public class AccountDemoRecordType {
    public String id {get; private set;}
    private static AccountDemoRecordType instance;
    
    private AccountDemoRecordType(){
        id = Account.sObjectType
            .getDescribe()
            .getRecordTypeInfosByName()
            .get('Demo')
            .getRecordTypeId();
    }
    
    public static AccountDemoRecordType getInstance() {
        if(instance == null) {
            instance = new AccountDemoRecordType();
        }
        return instance;
    }
}
```

### 优化要点
1. 使用私有构造函数防止直接实例化
2. 添加静态实例变量和 getInstance() 方法
3. 实现懒加载机制，仅在首次调用时创建实例

## 单例模式的优势

1. **性能提升**：只创建一次实例，减少了内存分配和垃圾回收的开销
2. **资源节约**：在处理大量记录时尤其有效，可以显著减少 CPU 时间和堆内存使用
3. **一致性保证**：确保所有代码使用相同的实例，维护全局状态一致性
4. **简化代码**：通过集中管理实例，简化了代码结构和使用方式

## 注意事项

1. 在多线程环境中使用单例模式需要考虑线程安全性
2. 过度使用单例可能导致代码耦合度增加，应谨慎使用
3. 在 Salesforce 中，要注意单例的生命周期不会跨越多个事务

## 结论

通过应用单例模式，我们可以有效地优化 Apex 代码，提高性能，并更好地遵守 Salesforce 的限制。在设计大规模、高性能的 Salesforce 应用时，合理使用设计模式如单例模式是非常重要的。

单例模式不仅可以应用于记录类型的获取，还可以用于缓存配置数据、管理共享资源等场景。在 Salesforce 开发中，灵活运用单例模式可以帮助我们编写更高效、更易维护的代码。

> 记住设计模式是工具，而不是规则。始终根据具体情况和需求来决定是否使用单例模式，以确保它真正为你的应用带来价值。