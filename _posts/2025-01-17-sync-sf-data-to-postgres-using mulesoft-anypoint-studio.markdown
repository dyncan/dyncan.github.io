---
layout: post
title: "使用 MuleSoft 实现 Salesforce Change Data Capture (CDC) 数据同步"
subtitle: ""
date: 2025-01-17 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-mulesoft-salesforce.png"
catalog: true
tags:
  - MuleSoft
  - Postgres
  - Sync
  - Salesforce
---

本教程将指导您如何利用 MuleSoft 平台订阅 Salesforce 的 Change Data Capture (CDC) 事件，并将数据实时同步到外部 PostgreSQL 数据库。通过这个实践示例，您将学习到完整的数据集成流程。

## 概览

我们将完成以下核心任务：
1. 在 Salesforce 中配置联系人对象的 CDC 事件发布
2. 使用 MuleSoft Salesforce Connector 配置 Replay Channel Listener 以订阅这些变更事件
3. 在 Anypoint Studio 中处理 CDC 事件数据
4. 将数据同步写入 PostgreSQL 数据库

![img](/img/in-post/post-bg-mule-salesforce.png)

## 环境准备

在开始之前，请确保您已准备好以下环境：

- **Anypoint Studio**: MuleSoft 的官方开发环境 [安装链接](https://docs.mulesoft.com/studio/latest/to-download-and-install-studio)
- **Salesforce Developer Edition**: 免费的 Salesforce 开发环境 [注册链接](https://developer.salesforce.com/signup)
- **PostgreSQL 数据库**: 一个可访问的 PostgreSQL 实例

## 详细步骤

### Salesforce CDC 配置

1. 使用开发者账号登录您的 Salesforce 开发者组织。
2. 导航至设置页面：
   - 进入 `Setup` -> `Home`
   - 在搜索框中输入 _Change Data Capture_
   - 进入详情页面选择`联系人`对象，此处可以看到所有可发布 CDC 事件的对象列表

![img](/img/in-post/post-bg-cdc-contact.png)

选择并保存后，您就完成了联系人对象 CDC 事件的配置。当联系人记录发生变更时，系统将自动向事件总线发布相应的事件。

### MuleSoft 项目配置

启动 `Anypoint Studio`, 点击 `file`, 然后选择 `New` > `Mule Project`. 项目名字可以叫 `salesforce-cdc` 然后点击 `Finish`.

![img](/img/in-post/post-mule-new-project.png)
   
创建一个 global.xml 文件来存储你的 connector 配置。右键点击 src/main/resources 文件夹。选择 `New` > `File`.

![img](/img/in-post/post-bg-cdc-create-global-yaml.png)

文件名为 `global.xml` 然后 点击 `Finish`.

![img](/img/in-post/post-bg-cdc-global-yaml.png)

在 global.yaml 文件中，通过复制和粘贴以下内容，创建以下属性。

```yaml
salesforce:
  username: "salesforce@example.com"
  password: "xxx"
  token: "xxx"
  
postgres: 
  username: "peter.dong"
  password: "xxx"
  database: "xxx"
  url:  "jdbc:postgresql://xxx.compute-1.amazonaws.com:5432/database"
```

在 Anypoint Studio 右边的 `Mule Palette`, 添加 `Salesforce` 和 `Database` 组件。

![img](/img/in-post/post-bg-salesforce-db-drag.png)

然后点击 `Salesforce`, 将 `Replay channel listener` 拖拽到左边的 Flow 里。

![img](/img/in-post/post-bg-drag-replay-channel-listener.png)
![img](/img/in-post/post-bg-replay-channel-listener.png)

点击 `Global Elements` 配置相关全局的变量。

![img](/img/in-post/post-bg-global-elem-config.png)

点击 `Create` .

![img](/img/in-post/post-bg-gloabl-elem-create.png)

输入框搜索 `Prop`, 点击 `OK`.

![img](/img/in-post/post-bg-config-props.png)

在弹出的界面里，输入 `global.yaml`, 这么做的目的是让 Mule Application 知道配置的 properties 在 global.yaml 文件里。

![img](/img/in-post/post-bg-config-props-global-yaml.png)

接下来配置 `Salesforce Configuration`, 重复上面步骤，点击 `Create`, 搜索 **salesforce** 关键字，点击 `OK`.

![img](/img/in-post/post-bg-salesforce-mule-01.png)

根据下图所示内容填写对应的参数，Authorization URL 可以留空，因为我们是 Salesforce 开发者账号，配置默认为 login.salesforce.com.

![img](/img/in-post/post-bg-salesforce-mule-02.png)

点击 `Test Connection`, 测试账号是否可以正常连接 Salesforce, 连接正常，点击 `OK`.

![img](/img/in-post/post-bg-salesforce-mule-03.png)

接下来配置 `Database Configuration`, 重复上面步骤，点击 `Create`, 搜索 **database** 关键字，点击 `OK`.

![img](/img/in-post/post-bg-salesforce-mule-04.png)

`Connection` 选择 `Generic Connection`.

![img](/img/in-post/post-bg-salesforce-mule-05.png)

`JDBC Driver` 选择 `Add Maven dependency`, 搜索框输入 `postgresql`

![img](/img/in-post/post-bg-salesforce-mule-06.png)
![img](/img/in-post/post-bg-salesforce-mule-07.png)

然后在 Connection 里，填入对应的 Database 参数，点击 `Test Connection`, 验证数据库是否可以连接成功。

![img](/img/in-post/post-bg-salesforce-mule-08.png)
![img](/img/in-post/post-bg-salesforce-mule-09.png)

配置 `message flow`, 选中 `Replay channel listener`, 按照如图所示选择相应参数，本次示例只在联系人创建的时候触发，所以 Replay Option 选择的是 `ONLY_NEW`

![img](/img/in-post/post-bg-salesforce-mule-10.png)

将 `Logger` 组件拖出，目的用于记录 Inboud payload.

![img](/img/in-post/post-bg-salesforce-mule-11.png)

设置变量，类似 `Logger` 组件，将 `Set Variable` 拖出放在 `Logger`组件的后面

![img](/img/in-post/post-bg-salesforce-mule-12.png)

重复以上步骤，设置三个变量：
第一个变量可以设置如下，目的是在 Inbound change 事件中 sfdc_id 的值与 Salesforce 的 record id 对应。

```
Display Name: Set Salesforce Record Id
Name: sfdc_id
Value: payload.data.payload.ChangeEventHeader.recordIds[0]
```
![img](/img/in-post/post-bg-salesforce-mule-13.png)

第二个变量设置如图所示：

```
Display Name: Set Last Name
Name: last_name
Value: payload.data.payload.Name.LastName
```

![img](/img/in-post/post-bg-salesforce-mule-14.png)

第三个变量设置如图所示：

```
Display Name: Set Change Type
Name: change_type
Value: payload.data.payload.ChangeEventHeader.changeType
```

![img](/img/in-post/post-bg-salesforce-mule-15.png)

将 `Choice` 拖出，目的是判断当前的 CDC 事件是否为 **CREATE** .

![img](/img/in-post/post-bg-salesforce-mule-16.png)
![img](/img/in-post/post-bg-salesforce-mule-17.png)


设置判断条件：`vars.change_type == 'CREATE'`

![img](/img/in-post/post-bg-salesforce-mule-18.png)

将 `Database` 的 `Insert` 操作拖出放入第一个选项框中。

![img](/img/in-post/post-bg-salesforce-mule-19.png)

设置 sql 语句，和匹配的参数，插入数据库。

![img](/img/in-post/post-bg-salesforce-mule-20.png)

启动 Mule Application:

![img](/img/in-post/post-bg-salesforce-mule-21.png)
![img](/img/in-post/post-bg-salesforce-mule-22.png)

在 Salesforce 中创建一条 Contact 记录，Lasrt Name 为：**MuleSoft CDC**

![img](/img/in-post/post-bg-salesforce-mule-23.png)

我们在 MuleSoft Application 里可以看到 CDC 事件已经触发，并且 payload 日志已经打印出来 :

![img](/img/in-post/post-bg-salesforce-mule-24.png)

再来看下数据库更新情况，有一条新的记录产生，sfdc_id 为刚才在 Salesforce 创建的联系人记录。

![img](/img/in-post/post-bg-salesforce-mule-25.png)

## 总结

通过本教程，您已经成功实现了以下功能：
- Salesforce 联系人数据变更的实时捕获
- 使用 MuleSoft 进行数据处理和转换
- 将数据实时同步到 PostgreSQL 数据库

这个集成方案为实现系统间的实时数据同步提供了可靠的基础。您可以基于此框架扩展更多的集成场景，如添加其他对象的 CDC 监听、增加更复杂的数据转换逻辑等。