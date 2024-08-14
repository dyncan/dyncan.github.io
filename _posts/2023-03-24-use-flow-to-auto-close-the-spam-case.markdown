---
layout: post
title: "Use Case: 使用 Salesforce Flow 自动关闭标记为垃圾邮件的 Case"
subtitle: ""
date: 2023-03-24 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-salesforce-flow.jpeg"
catalog: true
tags:
  - Use Case
  - Flow
---

Salesforce Flow 是 Salesforce 平台上的一种流程自动化工具，它可以帮助用户自动化各种业务流程。在本文中将会介绍一个 Flow 的使用场景。

这个 Use Case 是基于在工作中遇到的实际情况而来。客服团队每天都会收到一些垃圾邮件，这些邮件占据了他们的时间和精力，影响了他们的工作效率。他们想将这些电子邮件地址存在一个垃圾邮件列表中。之后如果有来自这些地址的新的 Case, 他们希望自动将其标记为垃圾邮件并关闭该 Case, 为了解决这个问题，我决定采用 Salesforce Flow 来解决这个问题。

整理下需求：

- 可以让用户在 Case 中通过触发按钮来添加黑名单，然后将该邮件标记为垃圾邮件并存储到数据库里。
- 每当从被阻止的 Email 中创建一个 Case 时，该 Case 应被自动标记为垃圾邮件并设置为 Close 状态。

针对需求 1, 我们可以创建一个 **Screen-Flow** 来满足，然后我们创建一个自定义对象 `Blacklist_Email__c` 来存储这些垃圾邮件列表。这个对象有一个自定义字段 `Blocked_Email__c`.

针对需求 2, 我们可以使用 **Record-Triggered**, 但是 Flow 触发的对象是 EmailMessage, 而不是在 Case 上。原因是当我们使用 Email-To-Case 时，Case 和 EmailMessage 之间的建立连接是在 Case 的触发器执行完毕后建立的。如果 Flow 在 Case 上运行 ,它将无法找到原始电子邮件地址。

### Screen Flow 构建步骤

我们让用户在 Case 详情页面上点击一个 Quick Action 按钮，所以我们创建一个 Screen Flow.第一步是创建一个 recordId 变量，来获取当前的 Case 记录。

![img](/img/in-post/post-salesforce-flow-auto-close-case-01.png)

先使用 `Get Records` 元素检查是否用户的 Email 已经加入到了黑名单里：

![img](/img/in-post/post-salesforce-flow-auto-close-case-02.png)

如果当前的 Case 有 Email, 而这个邮件还没有加入邮件列表黑名单里，我们就会进入下一步，创建一个黑名单记录。

![img](/img/in-post/post-salesforce-flow-auto-close-case-03.png)

创建一个黑名单记录：

![img](/img/in-post/post-salesforce-flow-auto-close-case-04.png)

Screen flow 创建完成之后，可以在 `Setup` -> `Object Manager` -> `Case` -> `Buttons, Links, and Actions` 上创建一个 Flow Action.

### Record-Triggered Flow 构建步骤

我们需要将这个 Record-Triggered Flow 运行在 EmailMessage 对象上，原因我已经在文章开始的时候说明了，因为我们需要更新 Case 记录，所以这里选择了 After-Trigger.

![img](/img/in-post/post-salesforce-flow-auto-close-case-05.png)

检查当前 Case 的 邮件是否已经加入了黑名单了：

![img](/img/in-post/post-salesforce-flow-auto-close-case-06.png)

如果 👆 步骤中返回记录了，说明该 Case 的邮件已经加入黑名单了，这样我们就可以进行下一步，更新 Case 为 Close 状态。我们使用 `Decision` 来判断：

![img](/img/in-post/post-salesforce-flow-auto-close-case-07.png)

我们需要创建一个 Case Variable 来更新 Case: 

![img](/img/in-post/post-salesforce-flow-auto-close-case-08.png)

然后创建一个 `Assignment` 来为 Case 赋值，其中 Case 的状态为 `Closed`, 然后更新 `Description` 字段，这里不建议使用 Description 字段存 Case 关闭的原因，建议新建一个文本字段 `Close Reason` 来存，这里是为了方便测试。

![img](/img/in-post/post-salesforce-flow-auto-close-case-09.png)

调用 `Update Records` 更新 👆 赋值的 Case 对象：

![img](/img/in-post/post-salesforce-flow-auto-close-case-10.png)

### 测试

我们通过 Email-to-Case 功能生成一个 Case, 并将当前 Case 的邮件加入黑名单：

![img](/img/in-post/post-salesforce-flow-auto-close-case-11.png)
![img](/img/in-post/post-salesforce-flow-auto-close-case-12.png)

然后我们再次使用这个邮箱地址发一封同样的邮件，来看下会发生什么：

![img](/img/in-post/post-salesforce-flow-auto-close-case-13.png)

Case 的状态自动变更为 `Closed`.

### 总结

在本文中，我们介绍了一个使用 Salesforce Flow 自动关闭垃圾邮件 Case 的示例。通过使用 Salesforce Flow 解决问题，提高了工作效率，减少了错误率。如果你也遇到了类似的问题，那么使用 Salesforce Flow 可能是一个非常有效的解决方案。