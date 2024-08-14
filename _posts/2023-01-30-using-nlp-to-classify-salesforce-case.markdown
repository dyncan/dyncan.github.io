---
layout: post
title: "基于 Cohere(NLP 模型平台) 对 Salesforce Case 进行归类"
subtitle: ""
date: 2023-01-30 12:00:00
author: "Peter Dong"
header-img: "img/pg-post-nlp-salesforce.png"
catalog: true
tags:
  - Large Language Model (LLM)
  - NLP
---

我们使用 Salesforce 构建客户服务系统的时候，往往客服团队需要每天处理大量不同类型的 Cases, 那么根据 Case 类型的不同则需要不同的客服团队去处理，如果通过人工的方式对 Case 分门别类则会降低工作效率，降低客户的满意度，所以是否有一些方式可以基于现有的 Cases 数据建立数据模型，对之后新的 case 进行分析预测然后自动分类型，这样的话也能提高整个客服团队的服务效率。

Salesforce 平台提供了 [Einstein Case Classification](https://help.salesforce.com/s/articleView?id=sf.cc_service_key_concepts.htm&type=5), 它是 Salesforce Einstein 的一种机器学习功能，它能够自动将客户问题分类到正确的类别中。这可以帮助客服团队更快地解决客户问题，并提高客户满意度。Einstein Case Classification 通过自然语言处理和机器学习算法训练模型来识别客户问题的关键字和短语，并将其分类到正确的类别中。关于 Einstein Case Classification 功能的介绍以及如何使用，我会在另外一篇文章中做详细教程。

今天的教程主要是使用第三方的自然语言处理平台对 Salesforce 的 Cases 数据进行预测分析并分类。我们将使用 Patterns 平台来 Salesforce 获取数据，用 Cohere 平台基于获取的数据构建 Large Language Model(LLM), 最后对新的 Case 进行分析预测并将分类的数据类型更新至 Salesforce.

### 准备工作

- 一个 Salesforce 账户 (可以免费注册一个[开发者版本](https://developer.salesforce.com/signup)). 
- 一个 [Patterns](https://www.patterns.app/) 的账户。
- 一个 [Cohere](https://docs.cohere.ai/) 的账户，以及 API Key.


### 将 Salesforce 数据导入 Patterns

将 Salesforce 数据导入 Patterns 的目的是对已有数据进行训练，便于之后的数据预测。数据越多肯定预测越准确。使用 Einstein Case Classification 的话则需要至少 400 条记录的数据量来构建模型才有效。

#### 创建一个 APP

打开 [Patterns](https://studio.patterns.app/home) 并创建一个新的应用程序。

![img](/img/in-post/post-bg-nlp-salesforce-01.png)
![img](/img/in-post/post-bg-nlp-salesforce-02.png)

创建完 App 后，进入一个面板界面，点击导航栏中的 `Search Components`,打开 Marketplace，找到一个名为 `Query Salesforce (SOQL)` 的节点，点击 **Add** 按钮。

![img](/img/in-post/post-bg-nlp-salesforce-03.png)

添加完组件之后，连接 Salesforce org, 点击 `connection`, 染后点击 `Connect to Salesforce`来添加一个新的 ORG.

![img](/img/in-post/post-bg-nlp-salesforce-04.png)

设置 SOQL 查询语句获取数据：

```
SELECT Id, Type, Subject, Status
FROM Case
WHERE Type != null
AND Status = 'Closed'
```

![img](/img/in-post/post-bg-nlp-salesforce-05.png)

重命名 output table 为 `salesforce_training`

![img](/img/in-post/post-bg-nlp-salesforce-06.png)

#### 配置 Webhook

Patterns 能够从 webhook 触发 graph 节点。但是 Salesforce 并不支持 webhooks，不过可以用 Apex 来解决这个问题。

添加一个 `Webhook` 组件：

![img](/img/in-post/post-bg-nlp-salesforce-07.png)

打开 webhook 组件并复制 URL(我们稍后会需要这个)

![img](/img/in-post/post-bg-nlp-salesforce-08.png)

#### 配置 Salesforce

添加 Remote Site URL: **Setup** > **Security** > **Remote Site Settings** , 添加 URL `https://api-production.patterns.app`

新建一个 Class, 当有新的 Case 创建的时候用于触发 Patterns 的 Webhook.

```java
public class Patterns
{
    @future(callout=true)
    public static void sendToPatterns(Set<Id> caseIds) {
        List<Case> cases = [SELECT Id, Subject FROM Case WHERE Id IN: caseIds];

        HttpRequest request = new HttpRequest();
        request.setEndpoint('https://api.patterns.app/api/v1/webhooks/your-webhook');
        request.setMethod('POST');
        request.setHeader('Content-Type', 'application/json');
        request.setBody(JSON.serialize(cases));

        Http http = new Http();
        http.send(request);
    }
}
```

创建一个 Apex 触发器，在 Case 创建时执行。

```java
trigger CaseClassifier on Case (after insert) {
    if(!Trigger.new.isEmpty()) {
        Patterns.sendToPatterns(Trigger.newMap.keySet());
    }
}
```

#### 添加 Python 组件

添加此组件的目的是：获取外部系统进入的数据并解析，然后将数据写入 table 中。

![img](/img/in-post/post-bg-nlp-salesforce-10.png)

#### 配置 Cohere

在搜索框中搜索 `Cohere Classify`, 并配置如下，我们本次 Demo 会自动填充 Case 的 Type 的值，其基于客户发送的 Subject 内容自动预测。

- **Example Category Column** = Type
- **Example Text Column** = Subject
- **Input Text Column** = Subject
- 将 `cohere_input` 连接到 `salesforce_testing` table
- 将 `cohere_examples` 连接到 `salesforce_training` table

![img](/img/in-post/post-bg-nlp-salesforce-11.png)

#### 训练数据模型

现在都准备好了，可以运行我们的 flow 了。假设你的 Salesforce 已经有我们可以训练的 Case 数据，把鼠标悬停在如图的节点上，点击 "run" 按钮，然后将我们的训练数据发送到 Cohere.

![img](/img/in-post/post-bg-nlp-salesforce-12.png)

#### 创建 Case 测试数据预测模型

在 Salesforce 中创建一个 Case, Subject 为：`Having issues with power generation` , 其他填入必填字段，我们发现 Type 字段为空值。

![img](/img/in-post/post-bg-nlp-salesforce-13.png)

保存之后，听过 Apex 代码触发 Webhook, patterns 将数据同步至 Cohere, 然后 Cohere 基于训练的数据模型预测出 Case 是属于哪种类型的，我们发现对 Subject 为 `Having issues with power generation` 的 Case 进行了预测，预测类型为 `Electrical`,可信度为 **0.965088**.

![img](/img/in-post/post-bg-nlp-salesforce-14.png)

最后通过 Python 代码，将预测的数据更新至 Salesforce:

![img](/img/in-post/post-bg-nlp-salesforce-16.png)
![img](/img/in-post/post-bg-nlp-salesforce-15.png)

### 总结

通过利用 Patterns 和 Cohere, 我们能够使用 LLM 建立出一个简单但强大的 Salesforce Case 分类方案。随着我们收集更多的数据，我们可以通过手动重新运行 graph 或用 cron job 调度它来不断微调我们的 LLM 模型。使我们的数据预测更加准确，这个 Demo 的意义是不仅仅帮助我们对 Salesforce Case 进行分类，更重要的时候我们开始使用人工智能解决方案改善我们的工作和生活。

