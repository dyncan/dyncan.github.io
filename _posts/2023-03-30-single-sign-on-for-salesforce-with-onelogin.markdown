---
layout: post
title: "使用 OneLogin 实现 Salesforce 的 Single Sign-On(SSO)"
subtitle: ""
date: 2023-03-30 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-salesforce-sso.png"
catalog: true
tags:
  - Single Sign-On
  - OneLogin
---

在现在的企业中常常需要使用多个应用程序来完成其日常工作, 而在这些应用中进行登录和身份验证可能会变得非常繁琐. 为了改善这种情况, 企业通常会实现单点登录(SSO)功能, 这使得用户只需一次登录即可访问所有应用程序.

在本文中, 我们将探讨如何使用 Salesforce 和 OneLogin 之间的 SSO 集成来简化企业的身份验证流程.

### 什么是 SSO ?

SSO(Single Sign-On)是一种身份验证机制, 允许用户使用一个用户名和密码登录多个相关但独立的软件系统. 它消除了需要在每个单独的应用程序中输入不同的凭据, 并提供了更好的用户体验.通过SSO, 用户只需登录一次, 然后就可以无缝地访问所有经过授权的应用程序, 而无需再次输入他们的身份凭证. 这种身份验证机制通常由企业或组织中的IT部门实施, 以简化用户访问和管理的复杂性.

### 为什么使用 SSO?

在现代企业中, 通常需要使用多个应用程序来支持业务流程. 例如: 一个企业可能会使用电子邮件客户端,文档编辑器, 项目管理工具和 CRM 系统等各种应用程序.

在这种情况下, 用户必须在每个应用程序中单独进行身份验证登陆,这样容易导致以下问题:

- **认证繁琐**: 如果用户需要经常登录不同的应用程序,他们可能需要记住多个用户名和密码,从而使身份验证变得更加复杂和耗时.

- **安全风险**: 如果用户使用相同的凭据(用户名和密码)来访问多个应用程序, 则一旦某个应用程序被攻击或数据泄露, 所有其他应用程序都面临安全风险.

- **用户体验差**: 频繁的身份验证登陆流程可能会影响用户体验, 并降低工作效率.

单点登录(SSO)是一种解决上述问题的技术, 它允许用户通过一个中央身份验证机制仅需一次登录即可访问多个应用程序. 具体地说, 当用户登录到其计算机或设备时, 他们只需输入自己的用户名和密码一次, 即可访问所有已配置为使用该SSO解决方案的应用程序.

因此, 使用 SSO 可以简化用户的身份验证流程, 提高安全性并提高工作效率. 它使得企业能够更好地管理其身份验证系统, 同时也可以降低IT支出和管理成本.

### 准备工作

`OneLogin` 账户注册: [https://www.onelogin.com/free-trial](https://www.onelogin.com/free-trial), 注册需要使用 Business Email, 如果想以个人身份注册的话, 最好使用一些不常见或者说不知名邮箱服务注册.

![img](/img/in-post/post-salesforce-sso-01.png)

`Salesforce 开发者账号`: [https://developer.salesforce.com/signup](https://developer.salesforce.com/signup)

### 使用 OneLogin 配置单点登录

账号注册完毕之后, 登陆到 Home Page之后, 点击 `Application` 进入, 点击 `Add App`

![img](/img/in-post/post-salesforce-sso-02.png)

搜索 _Salesforce_ 关键字, 选择可以使用 `SAML 2.0` 的 Connector.

![img](/img/in-post/post-salesforce-sso-03.png)

进入配置界面, 如有必要,可以编辑要显示的名称, 点击 `Save` 按钮.

![img](/img/in-post/post-salesforce-sso-04.png)

选择 `Configuration` tab, 找到 `Salesforce Login URL`, 该 URL 采用 `https://login.salesforce.com?so=<您的组织ID>` 的形式. 如果您不确定您的Salesforce 组织ID, 请到Salesforce中 `Company Information` 中查找, 点击 `Save` 按钮

![img](/img/in-post/post-salesforce-sso-05.png)

选择 `SSO` tab, 将 SAML 签名算法修改为 `SHA-256`, 点击 `Save` 按钮.

![img](/img/in-post/post-salesforce-sso-06.png)

选择 `SSO` tab, 点击 `View Details` 按钮, 然后点击下载, 这个文件会在后续的 Salesforce 中配置 SSO 中使用到.

![img](/img/in-post/post-salesforce-sso-07.png)
![img](/img/in-post/post-salesforce-sso-08.png)

### 在 Salesforce 中配置单点登录

登陆 Salesforce, 点击 `Setting`, 搜索 _Single Sign-On_ 关键字点击.

![img](/img/in-post/post-salesforce-sso-09.png)

点击 `Edit` 按钮, 修改配置开启 SAML SSO.

![img](/img/in-post/post-salesforce-sso-10.png)
![img](/img/in-post/post-salesforce-sso-11.png)

现在让我们创建一个 SAML SSO 的配置: 

![img](/img/in-post/post-salesforce-sso-12.png)

在SAML单点登录设置页面,按以下方式填写表格:

- **Name**: Salesforce-OneLogin
- **API Name**: SSO_OneLogin
- **Issuer**: 从 OneLogin 应用程序的 SSO tab 复制的 issuer URL.
- **Entity ID**: https://saml.salesforce.com
- **Identity Provider Certificate**: 单击 _Choose File_ 并上传你从 OneLogin 中的应用程序的 SSO Tab 下载的 `X.509 PEM `文件.
- **Request Signing Certificate**: 默认设置
- **Request Signature Method**: SHA-256
- **Assertion Decryption Certificate**: Assertion not encrypted
- **SAML Identity Type**: Assertion contains the Federation ID from the User object
- **SAML Identity Location**: Identity is in the NameIdentifier element of the Subject statement
- **Identity Provider Login URL**: 从 OneLogin 中你的应用程序的 SSO tab 复制 SAML Login URL
- **Identity Provider Logout URL**: 留空
- **Custom Error URL**: 留空
- **Service Provider Initiated Request Binding**: HTTP POST

参考 SSO 的配置:

![img](/img/in-post/post-salesforce-sso-13.png)

登陆 Salesforce, 点击 `Setting`, 搜索 _My domain_ 关键字点击, 在 `Authentication Configuration` 点击  `Edit` , 开启 SSO 登陆方式.

![img](/img/in-post/post-salesforce-sso-14.png)
![img](/img/in-post/post-salesforce-sso-15.png)

勾选 `SSO OneLogin` 选项:

![img](/img/in-post/post-salesforce-sso-16.png)

设置 `Federation ID`, 因为我们在刚才的 Salesforce SSO 配置中选择了 `Assertion contains the Federation ID...`, 所以在 SSO 登陆的时候账号需要和 Salesforce 用户的 Federation ID 匹配上.

找到 OneLogin 的用户详情界面, 复制 Username :

![img](/img/in-post/post-salesforce-sso-17.png)

在 Salesforce 中找到当前用户的详情信息, 编辑用户信息, 找到 `Federation ID` 填入刚才的值. 

![img](/img/in-post/post-salesforce-sso-18.png)

### 测试 SSO

我们使用 Chrome 的隐身模式, 打开 _https://coding-test-dev-ed.develop.my.salesforce.com_ (这里为本人 org 的测试地址), 这里需要换做你自己 org 的地址.

![img](/img/in-post/post-salesforce-sso-19.png)
![img](/img/in-post/post-salesforce-sso-20.png)

成功进入 Salesforce org:

![img](/img/in-post/post-salesforce-sso-21.png)

### 总结

使用 Salesforce 和 OneLogin 之间的 SSO 集成可以显著简化企业身份验证流程, 使用户能够更轻松地访问所需的应用程序. 通过遵循上述步骤, 您可以很容易地实现这种集成, 并确保安全地管理用户身份.





