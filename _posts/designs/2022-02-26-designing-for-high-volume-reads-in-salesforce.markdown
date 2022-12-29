---
layout: post
title: "在Salesforce中设计高吞吐量数据读取的一些想法"
subtitle: ""
date: 2022-02-26 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-systems-architect.jpeg"
catalog: true
tags:
  - Salesforce
  - High-Volume
  - Architecture
---

在使用 Salesforce 时,架构师学到的第一课是governor limits对性能的影响. Salesforce的多租户架构提供了巨大的灵活性和可扩展性, 正是它的共享性质决定了我们在使用Salesforce平台的同时也需要考虑的更多. 根据我们与 Salesforce 的集成方式,可以使用不同的技术来确保我们适用不同场景所需的业务规模.

随着API需求的增加, 出现可靠性问题的风险也在增加, 这可能会对企业产生影响. 了解围绕所有数据请求的业务需求很重要, 以确保请求得到及时响应. "及时"可以有不同的含义, 因为一些业务流程可能需要接近实时的数据, 而另一些则可以承受更大的延迟. 一旦清楚地了解了数据的业务需求, 就可以采用不同的策略来满足这些需求.

本文章探讨了当您正在架构的系统需要进行大量 API 调用以从 Salesforce 平台读取数据时需要考虑的问题.由于大批量读取和大批量写入的考虑因素略有不同, 大批量写入将在今后的文章中单独探讨.


#### 通过可扩展性来建立信任

当技术无法跟上企业的数据需求时,问题就不可避免地出现了.最常见的情况是速度减慢,用户在检索数据时遇到长时间的延迟.例如,在一个客户服务中心,糟糕的数据性能导致了更长的呼叫处理时间.但除此之外,甚至数据的质量也会受到影响.数据同步过程如果不够快,会导致分布式系统的数据不正确甚至损坏.

随着这些问题的增加, 对系统的信心和信任就会降低, 这可能会影响整个企业. 由此产生的混乱和沟通不畅会破坏企业用户和客户的信心. 也会导致客户流失, 以及更高的营业额支出.

为了建立信任, 你必须解决影响平台性能的可扩展性问题. 在解决这些问题之前, 你必须首先充分了解业务需求. 这种理解将使你能够在有必要进行折衷时做出明智的决定--即牺牲一个领域的性能来提高另一个领域的性能.

#### 扩展Salesforce的API

在构建与Salesforce的API集成时,最先想到的方案即使用标准的Salesforce REST和SOAP API.这两个API都支持大批量的读取,但要受到governor limits.这些限制随着时间的推移而改变,所以在设计一个可扩展的解决方案之前,一定要查看最新的[开发人员文档](https://developer.salesforce.com/docs/atlas.en-us.salesforce_app_limits_cheatsheet.meta/salesforce_app_limits_cheatsheet/salesforce_app_limits_platform_api.htm).对于API,两个关键限制是并发的API请求和总的API请求.你可以通过改变你的设计来绕过这些限制,但这些权衡会导致其他限制成为一个因素.

例如,你可以使用自定义的Apex REST API来整合一个请求,而不是提出多个独立的API请求.对几个不同的相关数据元素的请求可以被写成一个单一的请求.这种权衡减少了触及API请求总数限制的风险,但增加了其他限制的风险,如并发API请求限制,Apex CPU时间限制和Apex堆大小限制.

下面是一个简单的例子.针对原生Salesforce REST API依次进行了三次API调用.如果你开始遇到API总请求的限制,通过这个集成,你可以重新设计,使用一个单一的自定义REST API调用.

![img](/img/in-post/post-rest-api-01.jpeg)

getFullAccount API调用协调了Salesforce的数据,然后再将其返回给客户端.这将三个API调用减少到一个,有助于避免对API请求总数的限制.然而,你可以预期这一个调用将需要更多的时间来执行,可能会有并发API请求限制的风险.

![img](/img/in-post/post-rest-api-02.jpeg)

你也可以使用[Composite API](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_composite.htm) ,将多个相关的请求合并到一个调用中.这种方法简化了API调用,并减少了触及API请求总数限制的风险.从Winter '21开始,可以使用[Composite Graph API](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_composite_graph_introduction.htm)将一系列复杂的子请求打包成一个调用,允许你在一个single payload中处理多达500个子请求,并保证如果在一个特定的graph中任何部分操作失败,相关的事务会完全回滚.

#### Salesforce 流事件(Streaming Events)

Salesforce 提供了一个[Streaming event architecture](https://developer.salesforce.com/docs/atlas.en-us.api_streaming.meta/api_streaming/intro_stream.htm), 为处理大量数据提供了一种不同的方法. 与其从Salesforce同步请求数据, 不如将数据从Salesforce推送到其他系统. PushTopic, Change Data Capture(CDC), Platform 和 Generic Events 都为流式数据提供了略有[不同](https://developer.salesforce.com/docs/atlas.en-us.226.0.api_streaming.meta/api_streaming/event_comparison.htm)的功能. 您选择的事件类型将取决于您的具体用例, 但它们之间的总体架构模式是相似的.

例如,[Changed Data Capture](https://developer.salesforce.com/docs/atlas.en-us.change_data_capture.meta/change_data_capture/cdc_intro.htm) (CDC)事件提供了一种方法,可以在变化发生时通知外部系统Salesforce中的数据变化.因为CDC事件从根本上说是异步的,所以不能保证任何给定的更改会在外部系统上立即可用.然而,通过向外部系统提供数据并使用这些系统来处理数据请求,您可以减少直接从Salesforce读取大量数据的需要.

在这个例子中,一个Account记录被插入到Salesforce中.插入时,创建了一个CDC事件,该事件的订阅者可以对其作出反应.在这个例子中,一个外部订阅者获得了该事件的最新副本,并将该事件插入到一个外部数据库中.

![img](/img/in-post/post-rest-api-03.jpeg)

通过这种方法,其他系统可以根据外部数据库查询Account数据,而不需要直接访问Salesforce.

这种可扩展性优势确实伴随着一些复杂性.消息传递并不总是有保证的,所以在极少数情况下,Salesforce中的变化可能会丢失,导致与外部系统的同步问题.重要的是,要有一个程序来协调任何可能随着时间推移而出现的此类数据同步问题.即使有一个程序来协调,CDC事件也不能提供与同步交易方法一样的保证.然而,如果你的业务需求能在这些限制下得到满足,平台事件在可扩展性方面会有巨大的收益.

#### 利用Heroku进行扩展

作为Salesforce平台的一部分, Heroku很适合处理大量的API请求 ,并经常被用来提高可扩展性. 例如, 一个常见的模式是使用[Heroku Connect](https://www.heroku.com/connect)来实现Salesforce和Heroku Postgres之间的同步.

![img](/img/in-post/post-rest-api-04.jpeg)

大量的客户系统可以访问Heroku来检索与Salesforce保持同步的数据。随着客户规模的扩大，Postgres数据库的数量和Heroku dyno的数量都可以扩展，以满足增加的需求。同时，对Salesforce本身的需求仍然不受影响。

与platform events一样，这种可扩展性也会带来一些额外的复杂性。外部客户要么需要查询Postgres数据库，要么通过Heroku web dyno上实现的自定义API进行连接。Heroku Connect有强大的管理工具，但你需要考虑沙盒刷新如何与你的集成测试环境一起工作。另外，与platform events一样，在极少数情况下，数据会有一些不同步的风险。

安全是另一个需要考虑的因素，因为Salesforce的ownership结构并没有带入Heroku。如果你有复杂的安全要求，你可能在Heroku的实施中面临额外的复杂性。

对于许多企业来说，这种方法提供了两个最佳选择。Salesforce提供了一个灵活的基础架构，具有企业日常所需的稳定规模，而Heroku则通过扩大规模的能力来补充这一点，以满足大批量流程的需求，如果直接针对Salesforce运行，则会受到limits的影响。

#### MuleSoft

MuleSoft的Anypoint平台支持高度可扩展的集成，其架构与刚才描述的Heroku架构类似。都可以承担传入的读取请求的要求，并根据需要进行动态扩展。此外，Anypoint平台的工具简化了配置这些集成的大部分工作。

例如，为了支持大批量的读取，MuleSoft Anypoint可以在Salesforce数据前充当API网关。就其本身而言，这并不能解决大批量读取的问题，然而，Anypoint平台也提供了缓存功能。根据数据的性质，这种缓存能力可以大大减少对Salesforce的需求，同时最大限度地减少自定义代码。

![img](/img/in-post/post-rest-api-05.jpeg)

### 总结 

Salesforce提供了开箱即用的可扩展性，但作为一个共享系统，governor limits总是会产生一个性能上限。对于需要大批量读取的业务需求，你可能要考虑一个包括Heroku、MuleSoft或通过Salesforce流式事件更新的外部系统的架构.