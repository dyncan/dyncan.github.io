---
layout: post
title: "浅谈 Salesforce 的数据安全模型"
subtitle: ""
date: 2023-03-16 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-salesforce-data-security.png"
catalog: true
tags:
  - Data Security
  - Sharing Mode
---

Salesforce 的客户通常拥有几十到几百名担任不同职责的员工。Salesforce 的多层数据安全模型使管理员和应用程序开发人员能够从组织角度到个别记录层面管理这些员工对信息的访问权限。这意味着不仅可以提供更好的用户体验，还可以减少错误和提高安全性：如果用户账户被攻击，入侵者无法访问超出该用户权限范围的信息。

在 Salesforce 安全中，数据被存储在 3 个关键结构中：对象，字段和记录。Salesforce 安全的目标是对象级，字段级和记录级，对象类似于数据库中的表。字段类似于表中的列。记录类似于表内的数据行。Salesforce 使用对象级，字段级和记录级安全来确保对对象，字段和个人记录的访问。

## 数据访问和授权

在 Salesforce 中，Profiles, Permission Sets, Roles, 和 OWD settings 共同定义了用户可以访问和被授权在 Salesforce 中进行的操作.Permission Sets 是 Profile 的附加功能，为特定用户提供额外的权限.Roles 管理您可以看到的内容，而 Profiles 控制您可以执行的操作。

Salesforce 建议采用严格的数据访问策略。它使用 Profile 和 OWD 设置提供最少的权限和数据访问。此外，它通过使用其他访问权限，如共享规则和手动共享，扩大了访问权限，也可以通过配置来限制记录的可见性，比如：Restriction rule(Winter '22 Release), 它主要是根据一组规则和限制，旨在确保仅授权的用户才能访问和操作数据。

## Object-level-security

在允许用户访问记录之前，Salesforce 首先验证该用户是否有权限查看该记录的对象。对象级别的访问可以通过两种配置进行管理，即 Profiles 和 Permission Sets.

### Profiles

在 Salesforce 中，Profile 历来是控制对象级和字段级安全访问的方式。然而，自从 Permission Sets 发布后，我们建议使用它们作为配置对象和字段权限的主要方式，同时还有 Permission Sets Group. 请注意，给每个用户分配一个 Profile 仍然是必须的，因为这是你配置其他东西的地方，如页面布局分配或登录 IP 限制。然而，对于对象和字段的安全设置，配置最小访问量的 Profile, 并使用 Permission Sets 和 Permission Sets Group 来为用户增加权限。

![img](/img/in-post/post-salesforce-data-security-01.png)

### Permission sets 和 Permission Set Groups

推荐将 Permission sets 作为分配对象和字段权限的主要方式。因为这种方式更加灵活，更容易操作配置。与 Profile 相比，你可以为一个特定的用户添加多个 Permission sets. 这使你在设计安全模型时有更大的灵活性，因为功能可以以更细的方式分组。当只使用 Profile 时，你需要把所有的对象和字段的权限集中在一个 Profile 中，例如：一个销售主管。有了 Permission Sets, 你可以有不同的 Permission Sets 来代表销售主管执行的不同任务：管理线索，审批业务机会等，并随时向用户添加或删除较小的权限模块。

基于以上，在项目权限集的可能分的比较详细，在 Org 中会出现大量的 Permission Sets. 为了以更简单的方式管理它们，Salesforce 引入了 Permission Sets Group. 这样，将所有权限集归并在一个权限集的组里。方便为用户配置高效且准确的安全访问权限。


![img](/img/in-post/post-salesforce-data-security-02.png)

## Field-level-security

即使一个用户拥有对对象的访问权，她仍然需要对每个对象的各个字段进行访问。在 Salesforce 中，Profile 和 Permission Sets 也控制字段级的访问。管理员可以为单个字段提供读和写权限。管理员也可以将字段设置为隐藏，完全隐藏，从该用户那里取消对字段的访问。当你用 FLS 隐藏一个字段时，该字段将不能通过任何地方 (例如通过 API) 访问。推荐的安全最佳实践是使用字段级安全，而不是仅仅从记录页面或页面布局中删除一个字段。就像对象级安全一样，我们建议使用 Permission sets 和 Permission Set Groups 来配置字段级安全。

## Record-level security

记录级安全通常被称为 Salesforce 共享模式，或者只是简单的 "记录共享". Salesforce 提供了 5 种与他人共享记录和访问他人记录的方法。您可以从 owd 内的默认值开始，将您的数据锁定到最严格的级别。然后，您可以使用其他记录级别的安全工具，在需要时向选定的用户授予额外的权限。

![img](/img/in-post/post-salesforce-data-security-03.png)

### organization-wide sharing defaults

在 Salesforce 中，在记录中有一个叫做 `OwnerId` 的字段，指向一个真实的用户。记录的所有者通常是创建记录并对其有完全 CRUD 权限的人。Salesforce 提供了其他方法来自动将所有权分配给用户，以及将所有权从一个用户转移给另一个用户。

> 注意：所有权也可以被授予用户组，例如：Queues. 

组织范围默认设置 (OWD) 控制着每个对象 (例如：账户) 的每条记录被没有所有权的用户访问时的默认行为。

例如：

- 如果账户的 OWD 是 `private`, 这意味着 User 只能看到她是所有者的记录。
- 如果账户的 OWD 是`Public Read Only`, 意味着任何人都可以读 (但不能更新或删除) 该记录。
- 如果账户的 OWD 是 `Public Read/Write `, 这意味着任何人都可以读和更新 (但不能删除) 该记录。

但是能访问到记录的前提也是要结合 Profile/Permission Sets 的权限配置。二者相辅相成。请看下面的表格，列举了不同的 OWD 和 Profile 的配置决定了记录的不同访问权限：

| OWD               | Profile(CRED) | Result                                         |
|-------------------|---------------|------------------------------------------------|
| Private           | CRED          | My Records: Read/Edit                          |
| Private           | CR            | My Records: Read                               |
| Private           | -             | No Access                                      |
| Public Read Only  | CRED          | My Records: Read/Edit, Other Records: Read      |
| Public Read Only  | CR            | My Records: Read, Other Records: Read           |
| Public Read Only  | -             | No Access                                      |
| Public Read Write | -             | No Access                                      |
| Public Read Write | CRED          | My Records: Read/Edit, Other Records: Read/Edit |
| Public Read Write | R             | My Records: Read, Other Records: Read           |
| Public Read Only  | R             | My Records: Read, Other Records: Read           |
| Public Read Only  | CRED/View All | My Records: Read/Edit, Other Records: Read      |
| Public Read Write | CRED/View All | Read/Edit All Records                          |
| Private           | CRED/View All | My Records: Read/Edit, Other Records: Read      |
| Public Read Write | CR/View All   | My Records: Read, Other Records: Read           |
| Public Read Only  | CR/View All   | My Records: Read, Other Records: Read           |
| Public Read Only  | Modify All    | My Records: Read/Edit, Other Records: Read/Edit |
| Public Read Write | Modify All    | My Records: Read/Edit, Other Records: Read/Edit |
| Private           | Modify All    | My Records: Read/Edit, Other Records: Read/Edit |

### Role Hierarchies

通常在一个组织中，不同的角色有不同的记录访问要求。通常情况下，角色是按等级排序的：具有较高角色的用户应该能够访问较低角色的用户所能访问的记录。请注意，角色的层次结构很少映射到组织结构图。它真正映射的是数据访问的层次结构.Salesforce 提供了一个简单的方法来与管理人员共享记录，并代表一个角色层次结构。要使用这个共享规则，管理员必须首先将用户添加到一个角色。

### Sharing Rule

基于层级的分享只适用于向上和垂直方向的分享。如果我们想横向共享呢？例如，如果我们想把销售代表拥有的记录与她在销售团队中的同行分享，怎么办？这就是共享规则的作用。共享规则提供了一种横向共享记录的方法，让我们来了解一下细节。

#### Ownership-based sharing rules

基于所有权的共享规则让管理员根据角色，角色和下属以及 public group 的所有权来共享记录。例如，我们可以将销售主管角色中的任何人所拥有的所有记录与服务主管角色中的所有人共享。同样地，我们也可以与其他人共享销售主管及其下属所拥有的所有记录。

#### Criteria-based sharing rules

基于标准的共享规则让用户根据记录中某个字段的值访问记录，而不考虑谁拥有该记录。例如，如果用户 A 想看到在上海的所有账户，管理员可以设置一个基于标准的共享规则，与该用户共享所有城市为上海的账户。

### 手动共享 (Manual Sharing)

手动分享提供了一个与他人分享个人记录的机制。这个权限可以通过记录详情页上的 "共享" 按钮进入，让终端用户与他人共享个别记录。

> 注意：这只有在 OWD 是私有或公共只读的情况下才可用，因为否则 (比如公共读/写), 你也不需要用到它。

### Apex Sharing

在记录无法通过用户界面或设置进行共享的情况下，开发人员可以编写 Apex 代码，以编程方式进行共享。只有当记录的 OWD 被设置为私有或公共只读时才能使用。

### Restriction Rules

在上面讨论的共享模式中，为了控制 Salesforce 中记录的可见性，我们习惯于从最严格的设置开始 (即把对象的 OWD 设置为私有), 然后使用各种功能 (如角色等级，共享规则，团队等) 开放访问。但是，现在我们也可以定义一个规则来限制记录的可见性。例如：即使我们在一个对象上将 OWD 定义为公共读或公共读/写，我们现在也可以添加限制规则，从某些用户那里隐藏某些记录。

![img](/img/in-post/post-salesforce-data-security-05.png)
![img](/img/in-post/post-salesforce-data-security-06.png)

#### 限制性规则的支持对象

- Custom Objects
- Contracts
- Events
- Tasks
- Time Sheets
- Timesheet Entries

#### 在哪里适用限制性规则？

- List Views
- Lookups
- Related Lists
- Reports
- Search
- SOQL
- SOSL

#### 目前限制性规则的限制和考虑因素是什么？

- 在企业版中只能对支持的对象创建两个限制规则，在 Unlimited 和 Performance 版中只能创建五个限制规则。
- 限制性规则只支持 EQUALS 运算符。不支持 AND,OR 或任何其他运算符。
- 不支持使用公式。
- 在 RecordFilter 和 UserCriteria 字段中只支持以下数据类型。
  - Boolean
  - Date
  - DateTime
  - Double
  - Int
  - Reference
  - String
  - Time

### Implicit Sharing

除了以上这些比较熟悉的功能外，还有一些内置在 Salesforce 应用程序中的共享行为。这种共享被称为隐式共享，因为它不是由管理员配置的; 它是由系统定义和维护的，以支持销售团队成员，客户服务代表和客户或顾客之间的协作。请查看我的另一篇博客了解[隐式共享](https://dyncan.com/2023/03/28/implicit-sharing/).