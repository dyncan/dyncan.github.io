---
layout: post
title: "Salesforce DevOps Center 入门指南"
subtitle: ""
date: 2022-12-09 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-salesforce-devops-center.png"
catalog: true
tags:
  - DevOps
  - Guide
  - Salesforce
---

DevOps Center 是 Salesforce 在 2022 年 6 月发布的一个新工具。简单地说，这个工具允许你将元数据从一个 Salesforce Org 部署到另一个。例如，将自定义对象，Apex Class,Profiles 等从沙盒部署到生产。传统上，我们使用`Change Sets`来执行这项任务。

目前较为常用的针对 Salesforce 项目部署方案有如下几种：

  - DevOps Center
  - Salesforce CLI
  - VS Code with Salesforce Extensions & Salesforce CLI
  - ANT Migration Tool
  - Change Sets
  - 3rd party tools like Copado, GearSet etc.

### 使用 DevOps Center 的好处：

  - 使用 DevOps Center, 你可以在完全不相关的 Org 之间部署 metadata list. 比如从 Developer Edition Org 部署元数据到沙盒或生产环境，反之亦然。使用 Change Sets, 你只能在沙盒和生产环境之间部署变更的元数据。
  - DevOps Center 利用沙盒的`Source Tracking`功能，可以识别在开发环境中做出的修改。有了它，你就不需要在 excel 中跟踪所有改变的组件，然后手动添加它们。DevOps Center 将显示所有更改的组件，并让你选择要部署哪些组件。
  - DevOps Center 也支持元数据组件的`移除`. 这意味着，如果你在开发环境中删除一个自定义字段，它也将从其他环境中删除。传统上，我们需要使用 destructiveChanges.xml 来单独执行删除任务。
  - DevOps Center 使用 Source Control(Git). Source Control 维护着元数据组件中所有变化的完整历史。因此，如果项目在改变后不能像预期的那样工作，你可以立即比较这些变更历史并找到问题原因。

现在我们对 DevOps Center 有了一些了解，接下来我会介绍如何在 Salesforce 中使用它。

### Enable & Install DevOps Center

在 Salesforce 的 `Setup` 搜索 **DevOps** 关键词，然后启用 DevOps Center, 我目前是以我自己的 Develoepr Org 作为我自己的 DevOps Center Org, 你也可以在 Salesforce 的生产环境安装 DevOps Center Package.

![img](/img/in-post/post-bg-devops-center-guide-1.png)
![img](/img/in-post/post-bg-devops-center-guide-2.png)

### 创建 Connected App

在 Salesforce 的 `Setup` 搜索 **App Manager** 关键词，然后点击 `New Connected App`

![img](/img/in-post/post-bg-devops-center-guide-3.png)

根据下面图中的内容填入相对应的值，然后点击 `Save`.

![img](/img/in-post/post-bg-devops-center-guide-4.png)

保存之后，点击 `Manage`, 页面往下滚动，看到 `Permission Sets`, 点击 `Manage Permission Sets` 添加 **sf_devops_NamedCredentials**

![img](/img/in-post/post-bg-devops-center-guide-5.png)

![img](/img/in-post/post-bg-devops-center-guide-6.png)

### 为当前 User 分配 DevOps Center 权限

在 Salesforce 的 `Setup` 搜索 **User** 关键词，然后点击当前 User 进入用户详情页面，点击 `Edit Assignments`, 添加如图的对应的权限。

![img](/img/in-post/post-bg-devops-center-guide-7.png)
![img](/img/in-post/post-bg-devops-center-guide-8.png)

### 准备一个 GitHub Account

因为在使用 DevOps Center 的时候需要连接 GitHub 账号管理 branch, 所以如果没有 GitHub 账号的话需要注册一个，[注册链接](https://github.com/signup?ref_cta=Sign+up&ref_loc=header+logged+out&ref_page=%2F&source=header-home)

### 在 DevOps Center 创建一个 Project

在 Salesforce 的 `App Laucnher` 搜索 **DevOps** 关键词，然后点击进入。在第一次创建 Project 的时候会有一个弹框让你连接`Connect to GitHub`, GitHub 账号认证完成之后就可以创建 Project 了。

![img](/img/in-post/post-bg-devops-center-guide-9.png)
![img](/img/in-post/post-bg-devops-center-guide-10.png)
![img](/img/in-post/post-bg-devops-center-guide-11.png)

点击 `New Project`, 填入如下信息：

![img](/img/in-post/post-bg-devops-center-guide-12.png)

Project 在 DevOps Center 创建成功，这个创建过程可能会需要等一会。

![img](/img/in-post/post-bg-devops-center-guide-13.png)

Project 创建完成后，我们可以在 GitHub 上看到一个新的 repo 被创建了出来。

![img](/img/in-post/post-bg-devops-center-guide-14.png)

### 连接 Release 环境

现在让我们来连接 Release 环境。这里的环境指的是你要部署所有 Change 的生产环境。点击 `Click to Connect`

![img](/img/in-post/post-bg-devops-center-guide-16.png)
![img](/img/in-post/post-bg-devops-center-guide-17.png)

点击 `0 Dev / 1 Other` 连接开发和其他环境 (比如 Staing, UAT 等)

![img](/img/in-post/post-bg-devops-center-guide-18.png)

添加一个开发环境：

![img](/img/in-post/post-bg-devops-center-guide-19.png)
![img](/img/in-post/post-bg-devops-center-guide-20.png)
![img](/img/in-post/post-bg-devops-center-guide-21.png)

此次演示我只需要一个 UAT 环境，所以其他阶段的测试环境我将其移出掉不再使用。

![img](/img/in-post/post-bg-devops-center-guide-22.png)

连接 `UAT` 环境，点击 `Connect Environment...`, 注意我选择的是 Production Org, 我是将我的另外一个 Developer Org 作为我的 UAT 环境。

![img](/img/in-post/post-bg-devops-center-guide-24.png)
![img](/img/in-post/post-bg-devops-center-guide-25.png)

目前为止，我们所有的环境已经设置完毕，点击 `Activate`, 注意：Pipline 激活之后，你就无法再修改此 Pipline, 只能重新创建一个新的 Project, 重新配置一个新的 Pipline.

![img](/img/in-post/post-bg-devops-center-guide-26.png)

Pipline 现在已经激活，Pipline 的流程为：当你有一个需求需要部署的时候，你需要创建一个 Work Item, 一旦 Work Item 构建完成，在 Pipline 里可以将当前 Dev 环境的 Work Item 推送至 UAT 环境做 Users Testing, 一旦测试完成，可以在 Pipline 里将 UAT 的 Work Item 推送至生产环境。

![img](/img/in-post/post-bg-devops-center-guide-27.png)

DevOps Center 为我们创建了一个新的 UAT 分支。

![img](/img/in-post/post-bg-devops-center-guide-28.png)

Pipline 的目前的环境分布：

![img](/img/in-post/post-bg-devops-center-guide-29.png)

### 创建 Work Item

切换至 `Work Items`, 然后点击 `New Work Item`.

![img](/img/in-post/post-bg-devops-center-guide-30.png)
![img](/img/in-post/post-bg-devops-center-guide-31.png)

Work Items 列表：

![img](/img/in-post/post-bg-devops-center-guide-32.png)

点击进入 Work Item 详情，点击 `Proceed`

![img](/img/in-post/post-bg-devops-center-guide-33.png)

进入`In Process` 详情，如果你比较了解 Git 和 Source Control, 这一步类似创建一个 feature branch, 将我们的 work item 存入这个 branch 中。

![img](/img/in-post/post-bg-devops-center-guide-34.png)
![img](/img/in-post/post-bg-devops-center-guide-35.png)

现在让我们添加一些需要部署的组件，我们可以点击 `Pull Changes`将 Dev 环境所有的变化都拉取出来，也可以手动添加待部署的组件。我们点击`Add Components Manually`

![img](/img/in-post/post-bg-devops-center-guide-36.png)

添加待部署的组件，然后提交 commit:

![img](/img/in-post/post-bg-devops-center-guide-37.png)
![img](/img/in-post/post-bg-devops-center-guide-38.png)

通过`Activity History`, 可以看到提交历史：

![img](/img/in-post/post-bg-devops-center-guide-39.png)

在 GitHub 的 feature branch 可以查看到我们的提交记录：

![img](/img/in-post/post-bg-devops-center-guide-40.png)

### 完成 Review Process

目前我们已经完成了这个 Work Item, 现在可以将这些修改转移到 UAT.

![img](/img/in-post/post-bg-devops-center-guide-41.png)
![img](/img/in-post/post-bg-devops-center-guide-42.png)

点击 `View Change Request`, 我们会在 GitHub 上看到一个 PR 需要 Merge 到 UAT Branch

![img](/img/in-post/post-bg-devops-center-guide-43.png)
![img](/img/in-post/post-bg-devops-center-guide-44.png)

如果一切没有问题，我们可以点击 `Ready to Promote`.

![img](/img/in-post/post-bg-devops-center-guide-45.png)
![img](/img/in-post/post-bg-devops-center-guide-46.png)

### 部署组件到 UAT 环境

一旦 Work Item 的状态被标记为 `Ready to Promote`, 这个 Work Item 将出现在 Pipline 中，以便可以部署到其他环境：

![img](/img/in-post/post-bg-devops-center-guide-47.png)
![img](/img/in-post/post-bg-devops-center-guide-48.png)

点击 `Promote Selected`

![img](/img/in-post/post-bg-devops-center-guide-49.png)
![img](/img/in-post/post-bg-devops-center-guide-50.png)

现在，新建的 Object 已经部署到 UAT 环境了。

![img](/img/in-post/post-bg-devops-center-guide-51.png)
![img](/img/in-post/post-bg-devops-center-guide-52.png)

### 部署组件到生产环境

现在可以从 UAT 环境部署组件到生产环境了：

![img](/img/in-post/post-bg-devops-center-guide-53.png)

指定一个版本：
![img](/img/in-post/post-bg-devops-center-guide-54.png)
![img](/img/in-post/post-bg-devops-center-guide-55.png)

生产环境部署完成：

![img](/img/in-post/post-bg-devops-center-guide-56.png)

之前介绍过，我将自己的 Develoepr Org 作为生产环境，可以看到新增的对象已经部署到 Prod 环境：

![img](/img/in-post/post-bg-devops-center-guide-57.png)
![img](/img/in-post/post-bg-devops-center-guide-58.png)
![img](/img/in-post/post-bg-devops-center-guide-59.png)


