---
layout: post
title: "使用 User Mode 保护你的 Apex 代码"
subtitle: ""
date: 2022-11-21 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-apex-secure-coding.png"
catalog: true
tags:
  - User Mode
  - Salesforce
  - Feature
---

我们知道, 在默认情况下, Apex 代码以系统模式(`System Mode`)运行, 这意味着它具有比运行代码的用户更高的权限, 因此在执行代码的时候将忽略当前用户的权限, 这样一来, 即使运行的用户没有访问某个对象的权限, 但他们也能访问该对象. 如果应用程序构建不当, 这可能对于数据库的记录来说是一个安全风险. 有人可能会删除记录,即使他/她对该对象没有删除权限. 这篇文章将解释用户模式(`User Mode`)的访问级别权限, 这样可以增强 Apex 的安全上下文, 也将确保 Apex 代码操作的模式.

该功能处于Beta版中, 可以查看[Summer'22](https://help.salesforce.com/s/articleView?id=release-notes.rn_apex_UserMode_Database_Operations_Beta.htm&type=5&release=238)发布的内容详情.

在了解新的用户模式之前, 让我们看看在用户模式下运行 apex 代码有什么优势.在用户模式下,配置 Profile 级别的权限, Field-Level Security(FLS)和共享规则都适用于当前运行的用户. 因此, 如果我们在用户模式下运行 apex 代码, 那么它将尊重用户的权限和共享规则. 例如, 如果登录的用户没有访问某个对象或记录的权限, 那么他们将无法访问该对象. 他们在执行代码时将抛出异常. 而在系统模式下,它们由类的共享关键字控制.查看 [with sharing,without sharing 和 inherited sharing](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_classes_keywords_sharing.htm) 关键字.

### 系统模式和用户模式

下面是不同操作在不同模式下执行的操作列表.

| **System Mode**                                                                                      | **User Mode**                   |
|------------------------------------------------------------------------------------------------------|---------------------------------|
| Apex Class and Trigger                                                                               | Anonymous Apex                  |
| Apex Webservices                                                                                     | Chatter in Apex                 |
| Validation Rule, Auto Response Rule, Assignment Rule, Workflow Rule, Escalation Rule, Rollup Summary | Email Service                   |
| Approval Process, Publisher Action                                                                   | Standard Controller             |
| Test method without System.runAs()                                                                   | Test method with System.runAs() |
| Background or Async Jobs                                                                             |                                 |
| Flow called from Process Builder, Workflow, Custom Button, REST API                                  | Flow                            |


目前新的 Database 方法会接受一个新的 AccessLevel 类型参数,可接受 `USER_MODE` 或 `SYSTEM_MODE`. [参考文档](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_methods_system_database.htm#apex_System_Database_insert_5)

USER_MODE 下会检查以下内容:

  - Sharing Rules
  - Object Access
  - Field Access

### Apex Security Access 分类

Apex 默认不会强制执行对象和字段级别的权限.作为一个开发者,我们必须在 Apex 中强制执行 CRUD 和 FLS.


下面是关于字段和对象级安全的信息列表.

| **Record Level Security**            | **Object Level Security**                           |
|--------------------------------------|-----------------------------------------------------|
| WITH SECURITY_ENFORCED               | Schema Methods                                      |
| WITH USER_MODE ( beta阶段 )           | insert as user ( beta阶段 )                          |
| StripInAccessible                    | StripinAccessible                                   |
| Schema Methods                       | Database user mode operations ( beta阶段 ) |

#### Schema Methods

我们可以使用 `Schema.DescribeFieldResult` 来检查当前用户对某个字段是否有读取,创建或更新权限.

例如,如果我们想检查登录的用户对 Account 对象的 `PersonEmail` 字段是否有读取权限,我们可以把 SOQL 查询包含在一个 if 块中,该块使用上面描述的模式方法检查字段的访问权限.

```java
if (Schema.sObjectType.Account.fields.PersonEmail.isAccessible()) {
   Account c = [SELECT PersonEmail FROM Account WHERE Id= :Id];
}
```

#### WITH SECURITY_ENFORCED

`WITH SECURITY_ENFORCED` 可以在 SOQL 查询中使用,以强制执行字段和对象级别的安全权限.

对查询的 SELECT 语句中检索到的所有字段进行字段级权限检查.由于这个检测只在 SOQL 查询中起作用,它只在我们想检查某个字段的读取权限时有用.

```java
try
{
    List<Account> acts = [SELECT Id, Name, Email, 
                          (SELECT LastName FROM Contacts)
                        FROM Account 
                        WHERE Name like 'Apple' 
                        WITH SECURITY_ENFORCED];
} catch(System.QueryException){
    //TODO: Handle Errors
}
```

上述查询将返回Account的Id,Email和Name, 以及相关联系人的LastName, 前提是用户对这几个字段都有读取权限. 如果用户对这些字段中的至少一个字段的没有访问权, 查询将抛出一个System.QueryException异常, 并且不返回结果.

#### stripInaccessible

stripInaccessible 方法将在 Apex 中强制执行字段和对象级别的安全.这个方法将从sObject列表中移除当前用户没有权限的字段.

```java
List<Account> accts= new List<Account>{
    new Account(Name='Google', Logo__c='https://www.google.com/images/branding/googlelogo'),
    new Account(Name='Salesforce', Logo__c='https://www.salesforce.com/images/branding/salesforcelogo'),
};

// 移除不可创建的字段
SObjectAccessDecision decision = Security.stripInaccessible(
    AccessType.CREATABLE,
    accts);
try{
    // 获取用户有权限insert的字段
    insert decision.getRecords();
}catch(NoAccessException e){
    system.debug(e.getMessage());
}

// 打印移除的 fields
System.debug(decision.getRemovedFields());
```

#### User Mode in SOQL (Beta)

有了新的User Mode 来操作Database,我们可以在 SOQL 查询中指定用户模式.如果用户没有对该对象的 CRUD 访问权,那么代码将抛出一个错误.

让我们假设用户没有对下面对象的 CRUD 访问权限. 当我们在没有用User Mode的情况下运行下面的SOQL时, 它将无任何错误地执行.

```java
public static void testDefaultMode() {
    List<Customer__c> customers = [SELECT Id, Name FROM Customer__c];
    System.assertNotEquals(0, customers.size());
}
```

当我们登陆当前用户, 使用用户模式执行同样的 SOQL 时,代码将抛出一个错误.

```java
List<Customer__c> customers = [SELECT Id, Name FROM Customer__c WITH USER_MODE];
System.assertNotEquals(0, customers.size());
```

```log
System.QueryException: sObject type 'Customer__c' is not supported. If you are attempting to use a custom object, be sure to append the '__c' after the entity name. Please reference your WSDL or the describe call for the appropriate names.
```

#### User Mode for Static Query

```java
//Insert
Customer__c customer = new Customer__c();
customer.Name = 'Salesforce'
insert as user customer;

//Update
customer.Email__c = 'peter.dong@example.com'
update as system customer;

//Query
List<Customer__c> customers = [SELECT Id, Name, Email__c FROM Customer__c WITH USER_MODE];
```
#### User Mode for Dynamic Query

```java
//Insert 
Customer__c customer = new Customer__c();
customer.Name = 'Salesforce'
Database.insert(customer, AccessLevel.USER_MODE);

//Query
List<Customer__c> customers = Database.query('SELECT Id, Name, Email__c FROM Customer__c', AccessLevel.USER_MODE);

//Count
integer count = Database.countQuery('SELECT COUNT(Name) cnt FROM Customer__c', AccessLevel.USER_MODE);
```

### 用户模式的好处

用户将被限制性的访问Data, 没有 CRUD 权限, 他们不能做任何操作. 这将有助于减少数据的损失. 这也将有助于减少不正确的数据.













