---
layout: post
title: "聊聊 Salesforce 的 Implicit Sharing"
subtitle: ""
date: 2023-03-28 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-salesforce-implicit-sharing.png"
catalog: true
tags:
  - Data Security
  - Sharing
---

前几天我在 Discord 的 Salesforce 频道的讨论组里里看到一个人提出了一个关于 Salesforce 数据分享权限的问题. 这促使我想要分享一些有关该主题的知识. 今天我会讨论 Salesforce 关于隐式分享相关的知识点. 我希望这篇文章对所有对 Salesforce 权限感兴趣的读者都有所帮助. 另外我想表达的是 Salesforce 是一个非常强大和复杂的平台, 深入了解其权限系统可以帮助我们更好地管理和控制数据访问和业务流程.

![img](/img/in-post/post-salesforce-implicit-sharing-01.png)

先回答上面的问题: 如果一个用户能够访问 Opportunity, 该用户也能获得关联 Account 的 Read 权限(implicit parent sharing). 如果一个用户是某个 Account 的 Owner, 那么该用户能访问 opportunities 的前提需要看该用户的 Role 对 Opportunity 权限访问的设置(child implicit sharing).

## 隐式分享

Lightning 平台包括各种各样的共享功能, 管理员可以使用这些功能来明确授予个人或用户组的数据访问权. 除了这些比较熟悉的功能外, 还有一些内置在 Salesforce 应用程序中的共享行为. 这种共享之所以被称为隐式, 是因为它不是由管理员配置的, 而是由系统定义和维护的, 以支持销售团队成员, 客户服务代表和客户或顾客之间的协作.

### Parent Implicit Sharing

对拥有子记录访问权的用户来说,也拥有对父账户的只读访问权限. 还有一些需要注意的地方:

- 子记录对象的 OWD 需要设置为 private, 对设置为control by parent的子对象无效. 
- 只会影响到 Case, Contact 和 Opportunity 的账户.
- 只要用户对一个 Case, Contact 或者 Opportunity 记录有至少读的权限，用户就有对相关的 Account 读的权限。

#### Demo

**测试用户**: Peter Dong(Salesforce License)

**OWD Private:**Account, Case, Contact, Opportunity.

测试用户不能访问任何上面这些 private 权限的对象. 现在我将一个 Contact 记录分享给测试用户. 我们看会发生什么?

**测试前测试用户对 Contact 和 Account 访问权限:**

![img](/img/in-post/post-salesforce-implicit-sharing-02.png)
![img](/img/in-post/post-salesforce-implicit-sharing-03.png)

**将 Contact 的 Read/Write 权限分享给测试用户:** 我们发现测试用户可以访问 Contact 数据了也有编辑权限, 并且同时也能查看与之关联的 Account 数据.

![img](/img/in-post/post-salesforce-implicit-sharing-04.png)
![img](/img/in-post/post-salesforce-implicit-sharing-05.png)

我们现在通过测试用户修改 Account 数据, 会发现报错, 因为测试用户对 Account 只有 Read-only 的权限.

![img](/img/in-post/post-salesforce-implicit-sharing-06.png)

我们看下整体权限分配示意图:

![img](/img/in-post/post-salesforce-implicit-sharing-07.png)

### Child Implicit Sharing

如果某个用户是一个 Account 的 Owner, 那么该用户可以访问与之相关的子记录(只影响到 Case, Contact, Opportunity 对象), 另外需要注意的是, 也要看该用户的角色, 因为角色可以定义要提供哪些子记录访问权限.

![img](/img/in-post/post-salesforce-implicit-sharing-08.png)

#### Demo

**测试用户**: Peter Dong(Salesforce License)

**OWD Private:**Account, Case, Contact, Opportunity. 

期待结果: 将测试用户设置为一个 Account 的 Owner, 那么基于上面的 Role 的权限设置, 该用户应该可以访问 Case 和 Contact 和 Opportunity.

我们将 Account Owner 从 当前用户修改为测试用户:

![img](/img/in-post/post-salesforce-implicit-sharing-09.png)
![img](/img/in-post/post-salesforce-implicit-sharing-10.png)

我们来看下测试用户的 Opportunity List View, 发现可以看到 Opportunity Owner 不是测试用户的记录.

![img](/img/in-post/post-salesforce-implicit-sharing-11.png)

如何我们把 测试用户的 Role 对于 Opportunity 的权限修改为 `not access`:

![img](/img/in-post/post-salesforce-implicit-sharing-12.png)

我们再看下 Opportunity 的 List View, 发现测试用户已经无法查看到记录数据.

![img](/img/in-post/post-salesforce-implicit-sharing-13.png)

### Portal Implicit Sharing

门户网站隐式共享仅适用于 Partner Community License, 只影响已启用 partner community 的联系人.

#### Demo

**联系人**: Josh Davis(Partner Community License)

**OWD Private:**Account, Contact. 

我们通过 Josh Davis 用户登陆 Community:

![img](/img/in-post/post-salesforce-implicit-sharing-14.png)

查看 Account 和 Contact 记录, 我们发现可以查看到联系人和与之关联的 Account 记录.

![img](/img/in-post/post-salesforce-implicit-sharing-15.png)
![img](/img/in-post/post-salesforce-implicit-sharing-16.png)

如果我们在相同的 Account 下, 通过另外一个联系人新创建一个新的 partner uer, 联系人的数据如何呈现呢:

![img](/img/in-post/post-salesforce-implicit-sharing-17.png)

我们再通过 Josh Davis 用户登陆 Community, 查看 Contact 记录, 我们发现可以查看另外一个 Partner user 的联系人记录

![img](/img/in-post/post-salesforce-implicit-sharing-18.png)




