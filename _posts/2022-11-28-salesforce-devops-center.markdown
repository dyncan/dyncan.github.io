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



过去五年, DevOps 已逐渐成为Salesforce开发和项目发布管理的标准.从2022年12月开始,Salesforce的DevOps Center将在2022年12月发布General Availability(GA)版本, 这意味着 Salesforce 开发将更加符合其他平台上使用的实践,而不再受到Change Sets的限制.

### Salesforce DevOps Center 是什么?

DevOps Center是 Salesforce 在2022年6月发布的一个新产品/工具. 简单地说, 这个工具允许你将元数据从一个Salesforce Org部署到另一个. 例如, 将自定义对象,Apex类,Profiles等从沙盒部署到生产. 传统上,我们使用 "Change Sets" 来执行部署. DevOps Center结合了现代 DevOps的 最佳实践,其集中的用户界面简单,直观,易懂.

Salesforce DevOps Center可以让管理员和专业代码开发团队一起使用单一的配置和代码进行协作. 使每个人都能使用优雅的点击界面进行项目部署. DevOps Center对所有 Salesforce 客户免费提供, 旨在帮助每个组织实施 DevOps 最佳实践,包括Releases, User Stories, Pipelines, Work Items, and Source Control.

### 如何在Salesforce中使用 DevOps Center?

DevOps Center目前对拥有Salesforce专业版,企业版或无限版的用户是免费的,也可用于开发者版Org中.你也可以在Trailhead Playground或Scratch org中试用它. 您可以通过在Salesforce的Setup中访问DevOps Center.DevOps Center是一个托管包,需要安装到组织中.

>注意: 由于DevOps Center是一个管理包,所以它与Salesforce的每年三个版本发布无关,这给了团队更多的灵活性来启动它并提供更新.

![img](/img/in-post/post-bg-devops-center.png)

托管包安装完成后, 在开始使用DevOps Center之前, 还需要做一些配置工作, 包括: `Create Connected App`, `Assign DevOps Center Permission Sets to Users`. 具体配置请参考[官方文档](https://help.salesforce.com/s/articleView?id=sf.devops_center_setup.htm&type=5), 配置完成后, 在App Launcher里搜索 `DevOps Center`, 关于如何上手DevOps Center进行项目部署, 会在[另外一篇文章](https://dyncan.github.io/2022/12/09/step-by-step-guide-salesforce-devops-center/)中做详细教程, 本文不再介绍.

![img](/img/in-post/post-bg-devops-center-app.png)

![img](/img/in-post/post-bg-devops-center-new.png)

### Salesforce DevOps Center是如何工作的?

首先,让我们先了解一下Salesforce DevOps Center涉及的概念以及它们之间的关系.

#### Releases

任何metadata的变化都可以被归类为一个release.不管是一个大的release(包括上百个metadata组件)还是一个小的release变化(一个hotfix包括一两个字段或者class), 一个release可以被分割成一个或多个User Story.
 
#### User Stories

一个User Story定义了开发工作的一个范围, 以 Salesforce 项目为例, 不仅仅包括将要创建和更改的metadata内容, 它还包括了业务分析阶段获取的所有信息(比如: Architect diagrams, API Docs, ERDs, discussion notes等), 这将使开发工作更轻松准确地完成.这些附加信息将为上下文提供背景并消除由于沟通不畅而导致的重复沟通的问题. 

#### Work Items

在 Salesforce DevOps Center中, Work Items包含了需要部署的一些metadata list, 一个User Story可以创建并链接到一个或多个Work Item.通常情况下,它将关联到一个Work Item,但如果有一个Work Item的元数据项目被修改,那么就需要创建一个新的Work Item(因为你不能重新打开原来的Work Item), User Story是你看到这两个Work Item之间关系的地方,因为在 DevOps Cneter中是看不到的. 另外, 我们之前在org间部署通过Change Sets, 它实际上是一个release, 而在DevOps Center, 我们可以有多个Work Items,这些Work Items构成了一个发布, 因此,你可以将release分解成若干更小,更容易管理的Work Items

在DevOps Center, 当Work Items创建时, 你会把它链接到Salesforce的一个Org. 同时DevOps Center会创建一个 GitHub 分支.

![img](/img/in-post/post-bg-devops-center-work-items.png)

然后,你可以拉出该Salesforce org中所有元数据变化的列表,并(从列表中)选择你想添加到Work Item中的元数据.这比一次一次地建立元数据Change Sets要快得多. 

![img](/img/in-post/post-bg-devops-center-app-wi-progress.png)

#### Pipelines, Stages and Projects

Pipelines是Work items(即元数据变化)从开发到生产的流动.

一个Pipelines有一系列的阶段--每个阶段都与一个Salesforce org有关.一个阶段可以是用于开发的Scratch org或沙盒org.它可以是一个用于UAT或Staging的沙盒.而最后一个阶段是生产环境阶段.一个Work Items在Pipelines中从一个阶段 `promoted` 到下一个阶段.

![img](/img/in-post/post-bg-devops-center-app-pipelines-progress.png)

#### Source Control

DevOps Center 管理所有的GitHub分支,包括在分支之间merge元数据,以及Salesforce组织, Salesforce管理员不需要了解GitHub和CLI(DevOps Center屏蔽了这一点). 对管理员来说，Source Control也许是最大的变化--但所有的复杂性都被隐藏起来了.

#### DevOps Center局限性

DevOps Center是一个免费的部署工具，它的建立是为了取代Change Sets, 然而，DevOps Center并不是一个成熟的DevOps产品. 以下是DevOps Center目前的一些限制:

  - 目前 DevOps Center只与 GitHub 合作, 如果开发团队使用另一个源代码控制系统（例如 BitBucket），那么他们将需要更改为 GitHub，或者等待 DevOps Center的后续版本支持。
  - DevOps Center的方法目前支持基于组织的开发，而不是基于包的开发（DX）。基于组织的开发是目前Salesforce生态系统中绝大多数人采用的方法。
  - 没有与Jira（最流行的ticketing system）的整合，以保持Jira User Story与DevOps Center的Work Item的同步。
  - 当我们在一个pipline中部署metadata的时候, 无法查看元数据是否在不同的work item里, 这使得评估部署work item的风险变得困难.

