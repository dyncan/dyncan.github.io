---
layout: post
title: "探讨 Salesforce DevOps Center"
subtitle: ""
date: 2022-11-28 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-salesforce-devops-center.png"
catalog: true
tags:
  - DevOps Center
  - Salesforce
---

> DevOps Center is all about change and release management and introducing DevOps best practices to our entire community, regardless of where you fall on the low-code to pro-code spectrum – **Karen Fidelak, Salesforce DevOps Center Product Manager**



过去五年，DevOps 已逐渐成为 Salesforce 开发和项目发布管理的标准。从 2022 年 12 月开始，Salesforce 的 DevOps Center 将在 2022 年 12 月发布 General Availability(GA) 版本，这意味着 Salesforce 开发将更加符合其他平台上使用的实践，而不再受到 Change Sets 的限制。

### Salesforce DevOps Center 是什么？

DevOps Center 是 Salesforce 在 2022 年 6 月发布的一个新产品/工具。简单地说，这个工具允许你将元数据从一个 Salesforce Org 部署到另一个。例如，将自定义对象，Apex 类，Profiles 等从沙盒部署到生产。传统上，我们使用 "Change Sets" 来执行部署。DevOps Center 结合了现代 DevOps 的 最佳实践，其集中的用户界面简单，直观，易懂。

Salesforce DevOps Center 可以让管理员和专业代码开发团队一起使用单一的配置和代码进行协作。使每个人都能使用优雅的点击界面进行项目部署。DevOps Center 对所有 Salesforce 客户免费提供，旨在帮助每个组织实施 DevOps 最佳实践，包括 Releases, User Stories, Pipelines, Work Items, and Source Control.

### 如何在 Salesforce 中使用 DevOps Center?

DevOps Center 目前对拥有 Salesforce 专业版，企业版或无限版的用户是免费的，也可用于开发者版 Org 中。你也可以在 Trailhead Playground 或 Scratch org 中试用它。您可以通过在 Salesforce 的 Setup 中访问 DevOps Center.DevOps Center 是一个托管包，需要安装到组织中。

>注意：由于 DevOps Center 是一个管理包，所以它与 Salesforce 的每年三个版本发布无关，这给了团队更多的灵活性来启动它并提供更新。

![img](/img/in-post/post-bg-devops-center.png)

托管包安装完成后，在开始使用 DevOps Center 之前，还需要做一些配置工作，包括：`Create Connected App`, `Assign DevOps Center Permission Sets to Users`. 具体配置请参考[官方文档](https://help.salesforce.com/s/articleView?id=sf.devops_center_setup.htm&type=5), 配置完成后，在 App Launcher 里搜索 `DevOps Center`, 关于如何上手 DevOps Center 进行项目部署，会在[另外一篇文章](https://dyncan.github.io/2022/12/09/step-by-step-guide-salesforce-devops-center/)中做详细教程，本文不再介绍。

![img](/img/in-post/post-bg-devops-center-app.png)

![img](/img/in-post/post-bg-devops-center-new.png)

### Salesforce DevOps Center 是如何工作的？

首先，让我们先了解一下 Salesforce DevOps Center 涉及的概念以及它们之间的关系。

#### Releases

任何 metadata 的变化都可以被归类为一个 release.不管是一个大的 release(包括上百个 metadata 组件) 还是一个小的 release 变化 (一个 hotfix 包括一两个字段或者 class), 一个 release 可以被分割成一个或多个 User Story.
 
#### User Stories

一个 User Story 定义了开发工作的一个范围，以 Salesforce 项目为例，不仅仅包括将要创建和更改的 metadata 内容，它还包括了业务分析阶段获取的所有信息 (比如：Architect diagrams, API Docs, ERDs, discussion notes 等), 这将使开发工作更轻松准确地完成。这些附加信息将为上下文提供背景并消除由于沟通不畅而导致的重复沟通的问题。

#### Work Items

在 Salesforce DevOps Center 中，Work Items 包含了需要部署的一些 metadata list, 一个 User Story 可以创建并链接到一个或多个 Work Item.通常情况下，它将关联到一个 Work Item，但如果有一个 Work Item 的元数据项目被修改，那么就需要创建一个新的 Work Item(因为你不能重新打开原来的 Work Item), User Story 是你看到这两个 Work Item 之间关系的地方，因为在 DevOps Cneter 中是看不到的。另外，我们之前在 org 间部署通过 Change Sets, 它实际上是一个 release, 而在 DevOps Center, 我们可以有多个 Work Items，这些 Work Items 构成了一个发布，因此，你可以将 release 分解成若干更小，更容易管理的 Work Items

在 DevOps Center, 当 Work Items 创建时，你会把它链接到 Salesforce 的一个 Org. 同时 DevOps Center 会创建一个 GitHub 分支。

![img](/img/in-post/post-bg-devops-center-work-items.png)

然后，你可以拉出该 Salesforce org 中所有元数据变化的列表，并 (从列表中) 选择你想添加到 Work Item 中的元数据。这比一次一次地建立元数据 Change Sets 要快得多。

![img](/img/in-post/post-bg-devops-center-app-wi-progress.png)

#### Pipelines, Stages and Projects

Pipelines 是 Work items(即元数据变化) 从开发到生产的流动。

一个 Pipelines 有一系列的阶段--每个阶段都与一个 Salesforce org 有关。一个阶段可以是用于开发的 Scratch org 或沙盒 org.它可以是一个用于 UAT 或 Staging 的沙盒。而最后一个阶段是生产环境阶段。一个 Work Items 在 Pipelines 中从一个阶段 `promoted` 到下一个阶段。

![img](/img/in-post/post-bg-devops-center-app-pipelines-progress.png)

#### Source Control

DevOps Center 管理所有的 GitHub 分支，包括在分支之间 merge 元数据，以及 Salesforce 组织，Salesforce 管理员不需要了解 GitHub 和 CLI(DevOps Center 屏蔽了这一点). 对管理员来说，Source Control 也许是最大的变化--但所有的复杂性都被隐藏起来了。

#### DevOps Center 局限性

DevOps Center 是一个免费的部署工具，它的建立是为了取代 Change Sets, 然而，DevOps Center 并不是一个成熟的 DevOps 产品。以下是 DevOps Center 目前的一些限制：

  - 目前 DevOps Center 只与 GitHub 合作，如果开发团队使用另一个源代码控制系统（例如 BitBucket），那么他们将需要更改为 GitHub，或者等待 DevOps Center 的后续版本支持。
  - DevOps Center 的方法目前支持基于组织的开发，而不是基于包的开发（DX）。基于组织的开发是目前 Salesforce 生态系统中绝大多数人采用的方法。
  - 没有与 Jira（最流行的 ticketing system）的整合，以保持 Jira User Story 与 DevOps Center 的 Work Item 的同步。
  - 当我们在一个 pipline 中部署 metadata 的时候，无法查看元数据是否在不同的 work item 里，这使得评估部署 work item 的风险变得困难。

