---
layout: post
title: "探讨 Salesforce 中的 QA 自动化测试"
subtitle: ""
date: 2022-12-26 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-robot-framework-for-salesforce.jpeg"
catalog: true
tags:
  - CumulusCI
  - Robot-Framework
  - Salesforce
---

当 Salesforce 于 2000年2月份首次推出 CRM 服务时, 它只是一个用于存储客户信息和跟踪销售机会的数据库. 如今, 它几乎无人不晓, 因为从销售和市场营销到运营和财务的各种部门都依赖 Salesforce 来推动业务发展. 现在,每个 ORG 都有自己独特的工作流程,自定义对象和众多第三方集成,以满足其业务需求.这些定制优化了流程和功能,但在没有测试自动化的情况下部署这些功能到生产环境则会增加风险和降低交付质量, 并经常导致 UI 界面中的莫名的错误(明明之前功能没有问题?).

那么, 手动测试的缺点是什么?

  - 时间和成本高: 手动测试需要花费大量的时间来完成.
  - 易出错: 人为操作容易出现失误,导致测试结果不准确.
  - 效率低: 手动测试的效率很低,不能在短时间内完成大量的测试用例.
  - 难以重复: 手动测试难以重复执行,回归测试的话需要重新测试.
  - 难以跟踪: 手动测试难以跟踪测试结果,不易于统计和分析.

自动化测试的优点是什么:

  - 时间和成本低: 自动化测试可以在短时间内完成大量的测试用例,节省时间和资源.
  - 准确性高: 自动化测试可以确保测试结果的准确性.
  - 效率高: 自动化测试的效率高于手动测试.
  - 可重复性: 自动化测试可以轻松重复执行, 可以按需重新运行,并保证一致性.
  - 可跟踪性: 自动化测试结果可以方便的跟踪和统计, 易于分析和报告.
  - 可维护性: 自动化测试脚本可以长期维护, 不会因为人员流动而影响测试质量.

目前市场上也有许多的自动化工具, 但是大多缺乏 Salesforce 的支持, 今天介绍的这个自动化测试框架是完全[开源](https://github.com/SFDO-Tooling/CumulusCI)的, 也是由 Salesforce 官方团队推荐的.

### 框架介绍

CumulusCI 是一个基于 Python 的开源自动化测试框架, 专门用于 Salesforce 项目. 它提供了一组工具和框架, 用于简化 Salesforce 项目的持续集成和部署流程. CumulusCI 可以帮助用户自动化测试,部署,监控和管理 Salesforce 项目, 提高项目的效率和质量. CumulusCI 与 Salesforce DX, Jenkins 和 Git 等工具配合使用, 为 Salesforce 项目提供了完整的自动化测试和部署解决方案. 但是今天这篇文章,我将只涉及测试部分.

#### CumulusCI Robot Frameworks

CumulusCI Robot Frameworks 是一种在 CumulusCI 中使用的自动化测试框架,它基于 Robot Framework,一个通用的,开源的,可扩展的自动化测试框架.
CumulusCI Robot Frameworks 为 Salesforce 项目提供了一组用于执行自动化测试的关键字和库,它们可以帮助用户在 Salesforce 系统中进行自动化测试.

比如:

- 执行基本的 CRUD 操作
- 验证数据的正确性
- 执行高级的操作,如: 审批流程, 自定义对象和字段的测试
- 可以自动化测试 Apex 代码
  
CumulusCI Robot Frameworks 可以帮助用户编写可重复执行的,可维护的,易于理解和维护的自动化测试脚本,提高项目的质量和效率.

### 环境准备

在使用自动化框架测试之前, 需要配置一些电脑环境, 本文示例以 Mac 为主:

#### Python

推荐安装3.X.X版本的 Python(首选版本:3.9.X), 可以通过官方网站下载 [Python](https://www.python.org/downloads/macos/), 或者可以通过 [Homebrew](https://brew.sh/) 安装 Python 环境.

#### pip

pip 是 Python 的默认包管理器,用于安装和管理 Python 包(类似于 PHP 的 Composer, Java 的 maven, Ruby 的 Bundler). Python 版本 > `3.X` 时, 是默认安装 pip.

#### pipx

pipx 是一个用于管理独立的 Python 应用程序的工具.它主要作用是在不影响系统 Python 环境的情况下安装和管理 Python 应用程序.pipx 使用虚拟环境(venv)来管理应用程序,从而避免了应用程序之间的依赖冲突, 本示例中主要使用 pipx 来安装 CumulusCI 和相关依赖的库.

#### Salesforce CLI

通过[官方文档](https://developer.salesforce.com/docs/atlas.en-us.sfdx_setup.meta/sfdx_setup/sfdx_setup_install_cli.htm#sfdx_setup_install_cli_macos)安装和验证 Salesforce CLI 是否已经安装成功.

### 项目创建

这部分主要介绍如何使用 CumulusCI 创建一个自动化测试的项目.

#### 初始化项目

```
$ mkdir test-framework-demo 
$ cd test-framework-demo 
$ git init

$ cci project init // 使用此命令初始化cci项目
```

项目结构如下:

```
<ProjectName>
├── robot
│   └── <ProjectName>
│       ├── doc
│       ├── resources
│       └── tests
│           └── create_contact.robot
```

项目设置完毕, 可以将项目添加并提交到你的 Git 仓库.

```
$ git add .
$ git commit -m "Initialized CumulusCI project"
```

#### 配置测试环境

```
$ mkdir venv // 此文件夹主要用于隔离 Python 运行环境, 安装到其中的 Python 解释器,库和脚本与其他虚拟环境中的内容是隔离的.
$ python3 -m venv venv // 通过 Python 创建一个新的虚拟环境并安装在 venv 文件夹下
$ source venv/bin/activate // 激活虚拟环境
$ pip3 install robotframework-seleniumlibrary
$ pip3 install cumulusci
```

#### 连接 Salesforce ORG

连接生产环境或者开发者org:
```
$ cci org connect <org_name>
```

连接沙盒环境:
```
$ cci org connect <org_name> --sandbox
```

通过自定义登陆链接连接 org:
```
$ cci org connect <org_name> --login-url https://example.my.domain.salesforce.com
```

通过登录来验证与该org的连接是否成功:
```
$ cci org browser <org_name>
```

设置默认的org, 这一步是必须的, 因为要执行 robot 脚本, 需要先设置默认 ORG.

```
cci org default <org_name>
```

#### 执行自动化脚本

```
*** Settings ***

Resource        cumulusci/robotframework/Salesforce.robot
Library         cumulusci.robotframework.PageObjects

Suite Setup     Open Test Browser
Suite Teardown  Delete Records and Close Browser


*** Test Cases ***

Via API
    ${first_name} =       Get fake data  first_name
    ${last_name} =        Get fake data  last_name

    ${contact_id} =       Salesforce Insert  Contact
    ...                     FirstName=${first_name}
    ...                     LastName=${last_name}

    &{contact} =          Salesforce Get  Contact  ${contact_id}
    Validate Contact      ${contact_id}  ${first_name}  ${last_name}

Via UI
    ${first_name} =       Get fake data  first_name
    ${last_name} =        Get fake data  last_name

    Go to page            Home  Contact
    Click Object Button   New
    Wait for modal        New  Contact

    Populate Form
    ...                   First Name=${first_name}
    ...                   Last Name=${last_name}
    Click Modal Button    Save

    Wait Until Modal Is Closed

    ${contact_id} =       Get Current Record Id
    Store Session Record  Contact  ${contact_id}
    Validate Contact      ${contact_id}  ${first_name}  ${last_name}


*** Keywords ***

Validate Contact
    [Arguments]          ${contact_id}  ${first_name}  ${last_name}
    [Documentation]
    ...  Given a contact id, validate that the contact has the
    ...  expected first and last name both through the detail page in
    ...  the UI and via the API.

    # Validate via UI
    Go to page             Detail   Contact  ${contact_id}
    Page Should Contain    ${first_name} ${last_name}

    # Validate via API
    &{contact} =     Salesforce Get  Contact  ${contact_id}
    Should Be Equal  ${first_name}  ${contact}[FirstName]
    Should Be Equal  ${last_name}  ${contact}[LastName]
```

```
cci task run robot --suites robot/test-framework-demo/tests/create_contact.robot
```

执行结果:

![img](/img/in-post/post-bg-cci-salesforce-02.png)

测试结果日志:

![img](/img/in-post/post-bg-cci-salesforce-01.png)

#### 总结

本文主要介绍了如果通过自动化测试在Salesforce 标准 UI界面创建数据, 后面如果有时间的话, 会介绍如何针对定制化界面, 比如 lwc 组件, 审批流等进行自动化测试.



