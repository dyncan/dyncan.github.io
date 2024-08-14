---
layout: post
title: "在 VS Code 中使用 Apex Debugger "
subtitle: ""
date: 2023-01-28 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-debugging.png"
catalog: true
tags:
  - Debug
  - VS Code
---

在我们的日常开发中，调试 apex 的最常见做法是使用调试日志。然而，Salesforce 提供了对你调试代码很有用的一些其他方案。可以像 Java, Golang, PHP 那样通过断点调试自己的 code. 

`Apex Interactive Debugger`, 也叫 `Apex Debugger`, 允许客户使用 VS Code 作为客户端，在沙盒和 scratch orgs 中实时调试他们的 Apex 代码。你可以用它来：

- 在 Apex 类和触发器中设置断点。
- 查看变量，包括 sObject 类型，集合和 Apex 系统类型。
- 完成标准的调试操作，包括进入，结束和退出，以及运行到断点。
- 查看调用堆栈，包括由 Apex DML 激活的触发器，方法到方法的调用和变量。
- 把你的日志结果输出到 Debug Console.

但是如果使用 `Apex Interactive Debugger` 的话，需要 Salesforce org 有 `Apex Debugger Licenses`, 而这个 license 需要花钱购买，本次 Demo 是基于 developer org, 我会使用另外一个调试工具 `Apex Replay Debugger`. 它是一种免费的调试工具，可以帮助开发人员重放和调试 Apex 代码中的错误。它允许开发人员在不影响生产环境的情况下，重放和调试生产环境中发生的错误。这使得开发人员可以更好地了解错误的原因并解决问题。它与 Interactive Debugger 的区别是，Replay Debugger 需要基于日志文件调试，不是实时的进行断点调试。

以上涉及到的工具，都包含在 VS Code 中的[SF 扩展包](https://marketplace.visualstudio.com/items?itemName=salesforce.salesforcedx-vscode)里。

### 调试和测试代码

配置好 Apex Replay Debugger 扩展后，我们来做一些测试和调试，以修复一些 Apex 代码。

#### 部署代码到 org

先利用 Salesforce CLI 工具，创建一个项目，并认证一个 org, 然后分别创建一个 class 和一个测试类用于调试测试。

![img](/img/in-post/post-bg-apex-debug-01.png)

在 Visual Studio Code 中，右击文件夹 classes，然后选择 **SFDX: Deploy Source To Org**.

![img](/img/in-post/post-bg-apex-debug-02.png)

#### 运行 Apex Test Class

在 VS Code 中，使用快捷键 `Ctrl+Shift+P`(Windows 或 Linux) 或 `Cmd+Shift+P`(macOS) 来打开命令板。在搜索框中输入 **apex test**, 然后选择 **SFDX: Run Apex Tests**.

![img](/img/in-post/post-bg-apex-debug-03.png)

选择 `AccountServiceTest.cls`

![img](/img/in-post/post-bg-apex-debug-04.png)

注意输出面板中的测试结果.Apex 测试失败了！

![img](/img/in-post/post-bg-apex-debug-05.png)

错误信息表明，错误的值被分配给了账户的一个字段。让我们在代码中设置一个断点，重新运行测试先收集调试日志，然后重新调试日志以找到我们的代码错误。

#### 设置断点

调试时，断点 (breakpoint) 意味着正在运行的程序在特定行号处会暂停，以便开发人员可以检查该时间点的变量值。`Checkpoints` 是调试 Apex 代码的一项特殊功能，是一种断点，可通过捕获更多信息。您可以根据需要设置任意多个断点，但一次最多只能设置五个 Checkpoints. 与断点相比，Checkpoints 为所有局部变量，静态变量和触发上下文变量提供了更丰富的信息。为了区分断点和检查点，断点显示为一个实心红点，而 Checkpoints 显示为一个红色圆圈，中间有一条线穿过。

在 VS Code 中，打开 AccountService.cls 文件，将光标放在有 `return newAcct` 语句的一行。使用快捷键 `Ctrl+Shift+P`(Windows 或 Linux) 或 `Cmd+Shift+P`(macOS) 来打开命令板。在搜索框中输入 **sfdx checkpoint**, 然后选择 **SFDX: Toggle Checkpoint**.

![img](/img/in-post/post-bg-apex-debug-07.png)
![img](/img/in-post/post-bg-apex-debug-06.png)

使用快捷键 `Ctrl+Shift+P`(Windows 或 Linux) 或 `Cmd+Shift+P`(macOS) 来打开命令板。在搜索框中输入 **sfdx checkpoint**, 然后选择 **SFDX: Update Checkpoints in Org**. 你必须告诉 Salesforce 你的 Checkpoints，以便在你的 Apex 代码执行时收集堆信息。如果你修改了你的 Apex 代码或切换了 Checkpoints, 请再次运行此命令以保持同步。

![img](/img/in-post/post-bg-apex-debug-08.png)

#### 运行测试类并获取 Debug Logs

使用快捷键 `Ctrl+Shift+P`(Windows 或 Linux) 或 `Cmd+Shift+P`(macOS) 来打开命令板。在搜索框中输入 **sfdx replay**, 然后选择 **SFDX: Turn On Apex Debug Log for Replay Debugger**.

![img](/img/in-post/post-bg-apex-debug-09.png)

然后重新打开命令版，在搜索框中输入`apex test`,然后选择 **SFDX: Run Apex Tests**. 运行完毕，打开命令板。在搜索框中输入 **sfdx get**, 然后选择 **SFDX: Get Apex Debug Logs...**, 几秒钟后，会提示你选择一个要下载的调试日志。

![img](/img/in-post/post-bg-apex-debug-10.png)

#### 基于日志调试代码

在 VS Code 中，打开在上一步中下载的调试日志。你可以在 `.sfdx/tools/debug/logs` 文件夹中找到你用 VS Code 下载的调试日志。

![img](/img/in-post/post-bg-apex-debug-11.png)

右键单击调试日志中的任何一行，然后选择 **SFDX: Launch Apex Replay Debugger with Current File**. 几秒钟后，VS Code 会打开调试侧边栏，准备让你开始调试代码。

![img](/img/in-post/post-bg-apex-debug-12.png)

在调试工具栏上单击 "继续" 按钮，继续到第一个断点。如果你设置了多个断点，继续点击 Debug Toolbar 上的 Continue 按钮，直到调试器到达 AccountService.cls 中 `return newAcct` 语句。

![img](/img/in-post/post-bg-apex-debug-13.png)

在调试侧边栏中，展开 newAcct 变量，注意到 TickerSymbol 属性值 "SFDC" 与传递给 createAccount 方法的 tickerSymbol 参数值 "CRM" 不一致。

![img](/img/in-post/post-bg-apex-debug-14.png)

点击调试工具栏上的停止按钮，结束调试会话。

![img](/img/in-post/post-bg-apex-debug-15.png)

#### 修复代码并部署到 org

修复 AccountService.cls 中的代码，将 tickerSymbol 参数分配给 TickerSymbolfield. 保存文件，然后部署代码到 Salesforce org.

```java
TickerSymbol = tickerSymbol
```

重新运行测试类，并验证修复结果：

![img](/img/in-post/post-bg-apex-debug-16.png)




