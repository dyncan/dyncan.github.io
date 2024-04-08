---
layout: post
title: "如何为开源项目贡献 PR?"
subtitle: ""
date: 2024-04-08 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-guide-to-contributing-code-open-source-project.png"
catalog: true
tags:
  - Salesforce
  - Open Source
  - Git
  - GitHub
---

### 背景

博主一直从事 Salesforce 领域相关工作，常常利用 Salesforce 的 Chrome 扩展程序辅助工作以提高效率。在使用这些 Chrome 扩展程序的过程中，难免会遇到一些问题或者发现用户体验有待提升的情况。博主通常会在 GitHub 上寻找相关的开源项目，将其 Fork 到自己的 GitHub 仓库中，并尝试在本地修复问题，最后提交 PR 来改进项目。今天，我将分享自己在提交 PR 过程中的经历，以及参与开源项目可能遇到的一些问题，供大家参考。

### 如何提交 PR

其实我们大部分的操作是在命令行下进行的，有些朋友可能会使用集成在 IDE 中的 git 插件进行 PR。实际上，无论是使用命令行还是图形界面，其基本原理都是相同的。掌握了命令行，使用图形界面就更容易了。接下来，我将以 Salesforce 的 Chrome 扩展程序中比较知名的 [Salesforce-Inspector-reloaded](https://github.com/tprouvot/Salesforce-Inspector-reloaded) 扩展为例，谈谈如何提交 PR。接下来，我会解释为什么选择这个开源项目。

### 如何选择开源项目？

找到你想要贡献的项目后，让它符合以下标准，以确保它是一个不错的选择：

首先，你需要检查：

- 它有许可文件吗？
- 最新提交是什么时候完成的？是最近的吗？ （也就是说，项目是否在积极维护/工作？这个很重要）
- 维护者需要多长时间才能做出回应？回应及时吗？
- 是否对某个问题进行了积极的讨论？
- 最新的拉取请求最近多久合并？是最近吗？

如果上述条件都满足你的需求，那就去做吧，开始为这些项目做贡献。这也是博主选择 [Salesforce-Inspector-reloaded](https://github.com/tprouvot/Salesforce-Inspector-reloaded) 这个开源项目的原因。

### 提交 PR 的步骤

1. 找到原始项目，在右上角，点击 __Fork__, 将在你的 GitHub 账号下创建同一个项目的副本。仓库的 URL 将更改为：
```
https://github.com/dyncan/Salesforce-Inspector-reloaded
```

2. 在自己的仓库中，将 Fork 的项目 Clone 到本地
```
git clone git@github.com:dyncan/Salesforce-Inspector-reloaded.git
```

3. 在开始本地开发之前，务必先阅读项目的 `README` 文档，因为该文档可能包含有关如何贡献代码的重要说明。例如，在提交 PR 或修复 Bug 时，[Salesforce-Inspector-reloaded](https://github.com/tprouvot/Salesforce-Inspector-reloaded) 项目要求从 _releaseCandidate_ 分支创建新分支。此外，`README` 文档还会详细说明分支命名的规范，例如 `feature/my-new-feature` 或 `bugfix/xxx`。鉴于 Salesforce 的特殊性，您还需要准备一个 Salesforce developer org，以便在您对项目进行修改后运行其中的测试用例，以确保项目的现有功能不受影响。

4. 针对 [Salesforce-Inspector-reloaded] 项目，如果你的 Pull Request (PR) 是为了新增一个功能，请先在开源项目中提交一个功能请求 (Feature Request)。在页面上选择 `Pull requests` 选项卡，然后点击右侧的 `New pull request` 按钮。在 PR 的标题中清晰地描述你要添加的新功能，并在描述中详细说明该功能的目的和实现方式。这样可以让项目的维护者和其他贡献者对新增功能有充分的了解。

5. 在一切都准备好之后，就可以切换到一个新的分支，分支名最好紧贴这个更新的内容，比如 `feature/add-new-button`。
```
git checkout -b feature/add-new-button
```

6. 接下来，你需要进行必要的更改并提交这些更改。在进行更改和添加新文件之后，将这些更改添加到你创建的分支中。要查看你做的所有更改，请运行 git status 命令：
```
git status
```

7. 在你提交本地修改到远端仓库之前，需要将本地的测试用例部署到你的 Salesforce developer org 中，并确保所有测试用例都能通过。这一步很重要。
```
sf deploy metadata -d test/ -o [your-test-org-alias]
```

8. 在确保所有测试用例通过后，请在 `CHANGES.md` 文档中简要描述你所做的修改。我以之前提交的一个 PR 为例，其中 feature 351 是你之前创建的 Feature Request 编号。
```
Enhanced the user interface with a subtle inline "copied" indicator for field copy actions feature 351 (contribution by dyncan)
```

9. 然后就可以提交代码到远端仓库里。
```
git add .
git commit -m "feat: add new button"
```

10. 最后将新的分支推送到远程：
```
git push --set-upstream origin feature/add-new-button
```

11. 新建 PR
在您的 GitHub 仓库中找到相应的项目，进入`Pull requests` 标签页，点击 `New pull request` 按钮，创建一个新的 PR（Pull Request）。这样就发起了 PR，等待原项目成员审核并回复，您可以在原项目中查看状态。如果 PR 审核通过，项目状态将如下所示：

![img](/img/in-post/post-bg-pr-merged.png)

### 总结
参与开源项目不仅可以帮助项目变得更好，也可以提升自己的技能。通过对开源的贡献可以清晰地了解一个项目的未来，也可以学习新的编程技巧和最佳实践，建立良好的声誉，扩展专业网络，促进个人成长，为 Salesforce 社区做出贡献，这些都是非常有价值的。

