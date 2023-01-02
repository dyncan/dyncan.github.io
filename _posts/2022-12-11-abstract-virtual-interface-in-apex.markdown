---
layout: post
title: "Apex 中的 Abstract, Virtual, Interface"
subtitle: ""
date: 2022-12-11 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-oop.png"
catalog: true
tags:
  - OOP
  - Interface
  - Abstract
  - Virtual
  - Salesforce
---

> Object-oriented programming (OOP) is a programming paradigm based on the concept of "objects", which can contain data and code. The data is in the form of fields (often known as attributes or properties), and the code is in the form of procedures (often known as methods). - Wikipedia


### Interface

接口是面向对象编程语言中接口操作的关键字, 是一组相关方法的集合, 接口只能包含方法名,不能包含任何方法实现或成员变量.

#### 如何定义一个接口

```java
public interface MyInterface {}
```
#### 实现一个接口

```java
public class MyChildClass implements MyInterface {}
```

#### 接口的特点

  - 只有方法的定义, 没有方法的实现
  - 不能包含成员变量
  - 类可以实现多个接口(比如 Batch Class)

### Abstract

Abstract是面向对象编程语言中接口操作的关键字, 用该关键字修饰的类可以成为抽象类.

#### 如何定义一个抽象类

```java
public abstract class MyAbstractClass {}
```
#### 继承抽象类

```java
public class MyChildClass extends MyAbstractClass {}
```

#### 抽象类的特点

  - 可以有成员变量
  - 抽象类里可以包含 **abstract** / **virtual** 修饰的方法
  - 使用**abstract**修饰的方法子类继承后必须通过 **override** 实现该方法
  - class只能继承一个抽象类

### Virtual

Virtual修饰的Class称为虚拟类, virtual 修饰符声明了该class可以允许扩展和重写.

#### 如何定义一个虚拟类

```java
public virtual class MyVirtualClass {}
```
#### 继承一个虚拟类

```java
public class MyChildClass extends MyVirtualClass {}
```

#### 虚拟类的特点

  - 可以只包含 `virtual` 方法
  - 虚拟类可以被初始化
  - class只能继承一个虚拟类
  - 子类可以不用实现虚拟类的方法, 如果需要实现请使用 `override` 关键字

### 通过重构代码来理解三者的区别

让我们使用接口,抽象和虚拟类来重构简单的代码,我将使用工厂方法和抽象工厂模式.

#### 基础代码

```java
// DataFactory.cls
public class DataFactory {

    public static Account createAccount() {
        Account account = new Account(
            Name = 'My Account',
            Email = 'myAccount@email.com'
        );
        insert account;
        return account;
    }

    public static Contact createContact() {
        Contact contact = new Contact(
            LastName = 'My Contact'
        );
        insert contact;
        return contact;
    }
}
```

```java
// 调用
Account account = DataFactory.createAccount();
Contact contact = DataFactory.createContact();
```

#### Interface

```java
// DataFactory.cls
public interface DataFactory {
    sObject getRecord();
    sObject createRecord();
}
```

```java
// AccountFactory.cls
public class AccountFactory implements DataFactory {
    public sObject getRecord() {
        return new Account(
            Name = 'My Account'
        );
    }

    public sObject createRecord() {
        Account account = getRecord();
        insert account;
        return acccount;
    }
}
```

```java
// ContactFactory.cls
public class ContactFactory implements DataFactory {
    public sObject getRecord() {
        return new Contact(
            LastName = 'My Contact'
        );
    }

    public sObject createRecord() {
        Contact contact = getRecord();
        insert contact;
        return contact;
    }
}
```

```java
// DataCreator.cls
public class DataCreator {
    private final Map<sObjectType, System.Type> OBJECT_TO_FACTORY = new Map<sObjectType, System.Type>{
        Account.sObjectType => AccountFactory.class,
        Contact.sObjectType => ContactFactory.class
    };

    public static Object createRecord(sObjectType objectTypeToCreate) {
        DataFactory factory = (DataFactory) OBJECT_TO_FACTORY.get(objectTypeToCreate).newInstance();
        return factory.createRecord();
    }
}
```

```java
// 调用
Account account = (Account) DataCreator.createRecord(Account.sObjectType);
Contact contact = (Contact) DataCreator.createRecord(Contact.sObjectType);
```

#### Abstract

```java
// DataFactory.cls
public abstract class DataFactory {
    public abstract sObject getRecord();
    public sObject createRecord() {
        sObject record = this.getRecord();
        insert record;
        return record;
    };
}
```

```java
// AccountFactory.cls
public class AccountFactory extends DataFactory {
    public sObject getRecord() {
        return new Account(
            Name = 'My Account'
        );
    }
}
```

```java
// ContactFactory.cls
public class ContactFactory extends DataFactory {
    public override sObject getRecord() {
        return new Contact(
            LastName = 'My Contact'
        );
    }
}
```

```java
// DataCreator.cls
public class DataCreator {
    private final Map<sObjectType, System.Type> OBJECT_TO_FACTORY = new Map<sObjectType, System.Type>{
        Account.sObjectType => AccountFactory.class,
        Contact.sObjectType => ContactFactory.class
    };

    public static Object createRecord(sObjectType objectTypeToCreate) {
        DataFactory factory = (DataFactory) OBJECT_TO_FACTORY.get(objectTypeToCreate).newInstance();
        return factory.createRecord();
    }
}
```

```java
// 调用
Account account = (Account) DataCreator.createRecord(Account.sObjectType);
Contact contact = (Contact) DataCreator.createRecord(Contact.sObjectType);
```

#### Virtual

```java
// DataFactory.cls
public abstract class DataFactory {
    public abstract sObject getRecord();
    public sObject createRecord() {
        sObject record = this.getRecord();
        insert record;
        return record;
    };
}
```

```java
// AccountFactory.cls
public class AccountFactory extends DataFactory {
    public sObject getRecord() {
        return new Account(
            Name = 'My Account'
        );
    }
}
```

```java
// ContactFactory.cls
public class ContactFactory extends DataFactory {
    public override sObject getRecord() {
        return new Contact(
            LastName = 'My Contact'
        );
    }
}
```

```java
// BasicDataCreator.cls
public virtual class BasicDataCreator {
    public virtual Map<sObjectType, System.Type> getObjectToFactory() {
        return new Map<sObjectType, System.Type>{
            Account.sObjectType => AccountFactory.class,
            Contact.sObjectType => ContactFactory.class
        };
    }

    public static Object createRecord(sObjectType objectTypeToCreate) {
        DataFactory factory = (DataFactory) this.getObjectToFactory().get(objectTypeToCreate).newInstance();
        return factory.createRecord();
    }
}
```

```java
// AccountVariationAFactory.cls
public class AccountVariationAFactory extends DataFactory {
    public sObject getRecord() {
        return new Account(
            Name = 'My Variation A Account'
        );
    }
}
```

```java
// ContactVariationAFactory.cls
public class ContactVariationAFactory extends DataFactory {
    public override sObject getRecord() {
        return new Contact(
            LastName = 'My Variation AContact'
        );
    }
}
```

```java
// VariationADataCreator.cls
public class VariationADataCreator extends BasicDataCreator {
    public override Map<sObjectType, System.Type> getObjectToFactory() {
        return new Map<sObjectType, System.Type>{
            Account.sObjectType => AccountVariationAFactory.class,
            Contact.sObjectType => ContactVariationAFactory.class
        };
    }
}
```

```java
// 调用
BasicDataCreator basicDataCreator = new BasicDataCreator();

Account account = (Account) basicDataCreator.createRecord(Account.sObjectType);
Contact contact = (Contact) basicDataCreator.createRecord(Contact.sObjectType);

VariationADataCreator variationADataCreator = new VariationADataCreator();

Account accountVariationA = (Account) variationADataCreator.createRecord(Account.sObjectType);
Contact contactVariationA = (Contact) variationADataCreator.createRecord(Contact.sObjectType);
```
