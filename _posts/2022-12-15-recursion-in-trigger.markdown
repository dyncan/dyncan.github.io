---
layout: post
title: "Apex Trigger 中的递归调用现象"
subtitle: ""
date: 2022-12-15 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-recursion-trigger.png"
catalog: true
tags:
  - Design
  - Recursion
  - Trigger
  - Salesforce
---

递归 Trigger 对 Salesforce 开发人员来说是个大问题。我们总是希望在 Salesforce 中避免递归调用 Trigger. 但是，如果我们有一个庞大的功能复杂的 Org, 其中有多个带有父子关联的记录更新的触发器，包括 Process builder 和 Workflow field updates 等。那么我们就会很容易面临递归调用 Trigger 的问题。

因此，今天我将分享一些方法，使用这些方法我们可以轻松地避免递归触发器。

### 如何避免在 Trigger 中发生递归调用呢？

那么有几个不同的方法来解决触发器中的递归问题：

- 使用静态的 Boolean 变量控制
- 使用静态的 Set 集合去存储 Record Id.
- 使用静态 Map
- 使用 Old Map
- 遵循 Apex Trigger 最佳实践
  
让我们详细讨论以上解决方案和它们的局限性。

#### 使用静态的 Boolean 变量

你可以创建一个带有静态布尔变量的参数，其默认值为 true. 在触发器中，在执行你的代码之前 ,要检查该变量是否为真。代码执行完成，就把这个变量设置为 false,
但是这个方案有个问题：记录数量较少 (<200 条记录) 的话方案没问题。但是如果我们有大量的记录，它其实就无法工作。我们看如下代码：

Apex Class:

```java
public class ContactTriggerHandler{
     public static Boolean firstRun = true;
}
```

Trigger 代码：

```java
Trigger ContactTriggers on Contact (after update){
    Set<String> accIdSet = new Set<String>(); 
    if(ContactTriggerHandler.firstRun){
        ContactTriggerHandler.isFirstTime = false;
        System.debug('---- Trigger run ----> ' + Trigger.New.size() );
       // any code here
    }
}
```

在我的 ORG 中，有超过 200 条联系人，让我们查询所有联系人记录并尝试更新所有数据。在这种情况下，触发器应该执行 2 次。但是由于静态变量的原因，触发器只执行一次，并且会跳过剩余的记录。

```java
List<Contact> contLst =[SELECT ID, FirstName FROM Contact LIMIT 400];
update contLst;

System.debug('contLst size: ' + contLst.size());
```

结果打印：
```log
|DEBUG|---- Trigger run ----> 200
|DEBUG|contLst size: 360
```

查看 Salesforce 文档，了解 [Apex Trigger 的最佳实践](https://help.salesforce.com/s/articleView?id=000386331&type=1), 以避免在 Apex 类中使用静态布尔变量的递归。静态布尔变量是一种 **anti-pattern**, 所以请尽量不要使用。

#### 使用静态 Set

因此，最好使用一个静态 Set 或 Map 来存储所有已执行的记录 ID. 当下次再执行时，我们可以检查记录是否已经执行。

```java
public class ContactTriggerHandler{
     public static Set<Id> setExecutedRecord = new Set<Id>();
}
```

```java
trigger ContactTriggers on Contact (after update){
    System.debug('---- Trigger run ----> ' + Trigger.New.size() );
    for(Contact conObj : Trigger.New){
        if(!ContactTriggerHandler.setExecutedRecord.contains(conObj.id)){  
            // logic here
            // ...           
            ContactTriggerHandler.setExecutedRecord.add(conObj.Id);
        }    
    }
}
```

结果打印：
```log
|DEBUG|---- Trigger run ----> 200
|DEBUG|---- Trigger run ----> 160
|DEBUG|contLst size: 360
```

其实使用静态 Set 看起来也不是很理想，因为有很多时候，我们可能会在同一个事务中同时触发 insert 和 update(特别是混合使用 Apex 和 Flow 逻辑的情况下).

#### 使用静态 Map

我们可以使用静态 Map 来存储 Trigger 事件名称和 一组 Ids, 如下所示。

```java
public static Map<String,Set<Id>> mapExecutedRecord = new Map<String,Set<Id>>;
```

```java
public class ContactTriggerHandler {
    //on before update
    public static void beforeUpdate(map<ID, Contact> newMap, map<ID, Contact> oldMap) {
        List<Contact> conList = new List<Contact>();
        if(!ContactTriggerHandler.mapExecutedRecord.containsKey('beforeUpdate'))
            ContactTriggerHandler.mapExecutedRecord.put('beforeUpdate', new Set<ID>());
        for (Contact con: newMap.values()) {
            if(!ContactTriggerHandler.mapExecutedRecord.get('beforeUpdate').contains(con.Id)){
                conList.add(con);
                ContactTriggerHandler.mapExecutedRecord.get('beforeUpdate').add(con.Id);
            }
        }
    }
}
```

#### 使用 Old Map

在执行 Trigger 逻辑之前，总是使用 Old map 来比较值的变化。下面是我们如何比较旧值以解决触发器中的递归问题的例子。

```java
for(Opportunity opp: Trigger.New){
	if(opp.Stage != Trigger.oldMap.get(opp.Id).Stage){
		// logic here
	}
}
```
你的代码只会在值改变时执行。

### 总结

最后我们在使用这些方法的同时，还应该遵循以下最佳实践。

- **一个对象对应唯一一个 Trigger**: 这样的话，我们不必考虑 Trigger 的执行顺序。由于无法控制哪个 Trigger 会先被执行。
- **Trigger 里的逻辑尽量少**: 使用 Trigger 框架，然后在 Handler 里处理 具体的业务逻辑
- **处理递归 Trigger**: 为了避免触发器的递归，确保你的触发器只被执行一次。如果递归处理得不好，你可能会遇到错误："Maximum trigger depth exceeded'".



