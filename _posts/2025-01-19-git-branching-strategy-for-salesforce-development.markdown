---
layout: post
title: "Salesforce 开发的 Git 分支策略"
subtitle: ""
date: 2025-01-19 12:00:00
author: "Peter Dong"
header-img: "img/bg-post_git-branch-strategy.png"
catalog: true
tags:
  - Git
  - Branching
  - Salesforce
  - DevOps
---

在 Salesforce 开发过程中，一个合适的 Git 分支策略对于管理开发流程、修复 BUG 缺陷和跨环境部署至关重要。本文将介绍一个经过实践验证的 Git 分支策略，该策略不仅支持持续的功能开发，还允许支持团队独立处理问题修复，确保生产环境的稳定性和可靠性。

### 为什么需要分支策略？

- **版本控制**：管理不同版本的代码和配置
- **并行开发**：支持多个团队同时开发不同功能
- **风险控制**：隔离开发环境，保护生产环境
- **发布管理**：控制代码部署的节奏和质量
- **变更追踪**：便于审计和回溯

下图展示了 Git 分支策略的流程与 Salesforce 不同环境之间的关系：

<p align="center">
    <img src="/img/in-post/post-bg-git-branch-strategy.png">
</p>

### Feature 开发流程实践

#### 1. Development 分支同步（Rebase with Master）

在开始新功能开发之前，首先需要确保 `Development` 分支与生产环境代码保持同步。这一步骤确保所有开发都基于最新的生产代码进行。

```bash
# 同步Development分支
git checkout development
git fetch origin
git rebase origin/master
```

**注意：**应定期对 `Development` 分支进行 `Git Rebase` 操作，以同步 `Master/Main` 分支的最新更改。换句话说，每当有新代码合并到 `Master/Main` 时，需及时执行 `Rebase` 操作。

#### 2. 创建 Feature 分支

为新功能创建专门的 `Feature` 分支，确保开发工作相互隔离。

```bash
# 创建feature分支
git checkout development
git checkout -b feature/CRM-007-implement-batch-process
```

**命名规范：**
- 使用 feature/前缀
- 包含 Jira/工单编号
- 添加简短功能描述
- 使用小写字母和连字符

#### 3. 本地开发与测试

在 `Feature` 分支上进行开发，遵循 Salesforce 开发最佳实践。

**开发检查清单：**
- Apex 代码遵循规范
- 添加必要的注释
- 编写单元测试类
- 确保测试覆盖率达标
- 检查 SOQL 查询优化
- 验证触发器逻辑
- 审查权限设置

#### 4. 合并到 Development 分支

完成本地测试后，将 `Feature` 分支合并回 Development 分支。

```bash
git checkout development
git merge --no-ff feature/CRM-007-implement-batch-process
```

使用 `--no-ff`  命令可以保留开发分支的独立性，方便以后追踪功能的开发过程。

**合并前检查：**
- 代码审查完成
- 所有测试通过
- 文档更新完成
- 解决所有冲突

#### 5. QA 环境部署与测试

将代码合并到开发分支后，该分支将部署到 QA 环境进行测试。QA 专注于验证功能并识别与功能集成相关的任何问题。

**QA 测试重点：**
- 功能完整性验证
- 集成测试
- 性能测试
- 用户界面测试
- 自动化测试执行

#### 6. 创建 Release 分支

QA 测试通过后，创建 Release 分支准备 UAT 测试。

```bash
git checkout development
git checkout -b release/v1.1.0
```

**Release 分支规范：**
- 使用语义化版本号
- 只允许 bug 修复
- 不接受新功能
- 维护变更日志

#### 7. UAT 环境部署与测试

将 Release 分支部署到 UAT 环境进行用户验收测试。创建 Release 分支可以将经过测试的功能隔离，为下一次生产发布做好准备。

```bash
# UAT环境部署
sfdx force:mdapi:deploy -d deploy_package -u UATSandbox -l RunLocalTests
```

**UAT 测试清单：**
- 业务流程验证
- 用户体验评估
- 性能验收
- 数据迁移测试
- 集成系统测试

#### 8. 合并到 Master 并部署生产

UAT 通过后，将 `Release` 分支合并到 `Master/Main` 分支并部署到生产环境。

```bash
# 合并到 Master
git checkout master
git merge --no-ff release/v1.1.0
git tag -a 1.1.0 -m "Release version 1.1.0"
```

### Hotfix 开发流程实践

为什么需要独立的 Hotfix 流程？

在 Salesforce 开发中，生产环境的问题修复具有以下特点：
- 需要快速响应
- 不能影响正在进行的功能开发
- 需要确保修复的准确性
- 必须经过完整的测试验证
- 需要同步到所有相关环境

#### 1. 准备阶段：Hotfix 分支同步

首先，确保 Hotfix 分支与生产环境代码保持一致：

```bash
# 切换到 Hotfix 分支
git checkout hotfix

# 获取最新代码
git fetch origin

# 同步 Master 分支代码
git rebase origin/master
```

**Salesforce 特别注意事项：**
- 确认生产组织的最新部署状态
- 检查待修复组件的元数据版本
- 验证组织的自定义设置状态

#### 2. 创建修复分支

为每个问题创建独立的修复分支：

```bash
# 从 Hotfix 分支创建修复分支
git checkout hotfix
git checkout -b bugfix/CRM-567-resolve-duplicate-contacts
```

**分支命名规范：**
- 前缀：bugfix/
- JIRA CRM-XXXX
- 简短描述：描述问题本质

#### 3. 开发和本地测试

在修复分支上进行开发和测试。

**测试要求：**
- 单元测试覆盖率 ≥ 85%
- 包含正向和负向测试场景
- 验证边界条件
- 确保不影响其他功能

#### 4. 合并到 Hotfix 分支

确认修复有效后，合并回 Hotfix 分支：

```bash
git checkout hotfix
git merge --no-ff bugfix/CRM-567-resolve-duplicate-contacts
```

#### 5. 部署到 UAT 环境验证

使用 SFDX 命令部署到 UAT 环境：

```bash
# 转换源代码为部署格式
sfdx force:source:convert -d deploy_package

# 部署到 UAT 环境
sfdx force:mdapi:deploy -d deploy_package -u UAT --testlevel RunLocalTests
```

**验证清单：**
- 所有测试通过
- 问题已修复
- 无新的问题产生
- 性能指标正常
- 用户流程验证

#### 6. 合并到主分支

验证通过后，将修复合并到 Master：

```bash
git checkout master
git merge --no-ff hotfix
git tag -a v1.2.1 -m "Hotfix: Resolve Duplicate Contacts Fix"
```

#### 7. 生产环境部署

使用 Change Sets 或 SFDX 命令进行生产部署：

**部署注意事项：**
- 选择适当的部署窗口
- 准备回滚方案
- 监控部署过程
- 准备用户通知

#### 8. 同步到 Development 分支

确保修复同步到开发分支：

```bash
git checkout development
git merge --no-ff hotfix
```

### 总结

基于以上的 Git 分支策略的 Salesforce 开发流程，在保证生产环境稳定性的同时，实现了功能开发和问题修复的高效协同。团队也能够高效开发、快速响应问题并保持稳定可靠的代码库。