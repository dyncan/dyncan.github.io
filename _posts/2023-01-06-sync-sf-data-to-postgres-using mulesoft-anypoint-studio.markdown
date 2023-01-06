---
layout: post
title: "使用 MuleSoft Anypoint Studio 同步 Salesforce 数据同步至 Postgres"
subtitle: ""
date: 2023-01-06 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-mulesoft-salesforce.png"
catalog: true
tags:
  - MuleSoft
  - Postgres
  - Sync
  - Salesforce
---

在本教程中, 我们将学习如何从在 Salesforce 中订阅 Change Data Capture(CDC)事件然后利用 MuleSoft 平台将数据同步至外部数据库(本示例为 Postgres). 本教程主要包括:

  - 在 Salesforce 的联系人对象上发布一个 change data capture 事件
  - 使用 Salesforce Connector 设置 Replay Channel Listener 来订阅联系人变更事件
  - 在 Anypoint Studio 中查看 change data capture 事件的结构
  - 在 Postgres 数据库中查看同步的数据

![img](/img/in-post/post-bg-mule-salesforce.png)

在开始前需要准备的环境:

  - **Anypoint Studio**: 为 MuleSoft 开发者提供的开发环境, [点击安装](https://docs.mulesoft.com/studio/latest/to-download-and-install-studio)
  - **Salesforce Developer Edition Org**: 一个免费的 Salesforce 开发者环境. 注册请[点击](https://developer.salesforce.com/signup)
  - **Salesforce Security Token**: 一个区分大小写的字母数字密钥, 与密码结合使用, 以通过 API 访问 Salesforce 资源
  - **Postgres**: 一个关系型数据库, 因为需要连接数据库, 所以需要提供一个可访问的 PostgreSQL 数据库.

### Setup CDC in Salesforce

1. 使用您在注册时设置的用户名和密码登陆到您的 Salesforce 开发者版本 Org.

2. 打开 `SetUp` -> `Home`, 在搜索框中输入 **Change Data Capture**, 点击进入详情页, 这个页面显示了可以在 Salesforce 中发布 CDC 事件通知的可用对象, 在本教程中,我们将发布联系人对象的 CDC 事件.找到联系人对象,选择它.

![img](/img/in-post/post-bg-cdc-contact.png)

点击保存, 你现在已经指定,当一个联系人记录的数据发生变化时,一个事件将被发布到一个共享的事件 event bus.现在我们将设置一个 Mule project 来订阅这些事件.

### Create Mule project

启动 `Anypoint Studio`, 点击 `file`, 然后选择 `New` > `Mule Project`. 项目名字可以叫 `salesforce-cdc` 然后点击 `Finish`.

![img](/img/in-post/post-mule-new-project.png)
   
创建一个 global.xml 文件来存储你的 connector 配置。 右键点击 src/main/resources 文件夹。选择 `New` > `File`.

![img](/img/in-post/post-bg-cdc-create-global-yaml.png)

文件名为 `global.xml` 然后 点击 `Finish`.

![img](/img/in-post/post-bg-cdc-global-yaml.png)

在 global.yaml 文件中, 通过复制和粘贴以下内容, 创建以下属性.

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

在 Anypoint Studio 右边的 `Mule Palette`, 添加 `Salesforce` 和 `Database` 组件.

![img](/img/in-post/post-bg-salesforce-db-drag.png)

然后点击 `Salesforce`, 将 `Replay channel listener` 拖拽到左边的 Flow 里.

![img](/img/in-post/post-bg-drag-replay-channel-listener.png)
![img](/img/in-post/post-bg-replay-channel-listener.png)

点击 `Global Elements` 配置相关全局的变量. 

![img](/img/in-post/post-bg-global-elem-config.png)

点击 `Create` .

![img](/img/in-post/post-bg-gloabl-elem-create.png)

输入框搜索 `Prop`, 点击 `OK`.

![img](/img/in-post/post-bg-config-props.png)

在弹出的界面里, 输入 `global.yaml`, 这么做的目的是让 Mule Application 知道配置的 properties 在 global.yaml 文件里.

![img](/img/in-post/post-bg-config-props-global-yaml.png)

接下来配置 `Salesforce Configuration`, 重复上面步骤, 点击 `Create`, 搜索 **salesforce** 关键字, 点击 `OK`.

![img](/img/in-post/post-bg-salesforce-mule-01.png)

根据下图所示内容填写对应的参数, Authorization URL 可以留空, 因为我们是 Salesforce 开发者账号, 配置默认为 login.salesforce.com.

![img](/img/in-post/post-bg-salesforce-mule-02.png)

点击 `Test Connection`, 测试账号是否可以正常连接 Salesforce, 连接正常, 点击 `OK`.

![img](/img/in-post/post-bg-salesforce-mule-03.png)

接下来配置 `Database Configuration`, 重复上面步骤, 点击 `Create`, 搜索 **database** 关键字, 点击 `OK`.

![img](/img/in-post/post-bg-salesforce-mule-04.png)

`Connection` 选择 `Generic Connection`.

![img](/img/in-post/post-bg-salesforce-mule-05.png)

`JDBC Driver` 选择 `Add Maven dependency`, 搜索框输入 `postgresql`

![img](/img/in-post/post-bg-salesforce-mule-06.png)
![img](/img/in-post/post-bg-salesforce-mule-07.png)

然后在 Connection 里, 填入对应的 Database 参数, 点击 `Test Connection`, 验证数据库是否可以连接成功.

![img](/img/in-post/post-bg-salesforce-mule-08.png)
![img](/img/in-post/post-bg-salesforce-mule-09.png)

配置 `message flow`, 选中 `Replay channel listener`, 按照如图所示选择相应参数, 本次示例只在联系人创建的时候触发, 所以 Replay Option 选择的是 `ONLY_NEW`

![img](/img/in-post/post-bg-salesforce-mule-10.png)

将 `Logger` 组件拖出, 目的用于记录 Inboud payload.

![img](/img/in-post/post-bg-salesforce-mule-11.png)

设置变量, 类似 `Logger` 组件, 将 `Set Variable` 拖出放在 `Logger`组件的后面

![img](/img/in-post/post-bg-salesforce-mule-12.png)

重复以上步骤, 设置三个变量: 
第一个变量可以设置如下, 目的是在 Inbound change 事件中 sfdc_id 的值与Salesforce的 record id 对应.

```
Display Name: Set Salesforce Record Id
Name: sfdc_id
Value: payload.data.payload.ChangeEventHeader.recordIds[0]
```
![img](/img/in-post/post-bg-salesforce-mule-13.png)

第二个变量设置如图所示:

```
Display Name: Set Last Name
Name: last_name
Value: payload.data.payload.Name.LastName
```

![img](/img/in-post/post-bg-salesforce-mule-14.png)

第三个变量设置如图所示:

```
Display Name: Set Change Type
Name: change_type
Value: payload.data.payload.ChangeEventHeader.changeType
```

![img](/img/in-post/post-bg-salesforce-mule-15.png)

将 `Choice` 拖出, 目的是判断当前的CDC事件是否为 **CREATE** .

![img](/img/in-post/post-bg-salesforce-mule-16.png)
![img](/img/in-post/post-bg-salesforce-mule-17.png)


设置判断条件: `vars.change_type == 'CREATE'`

![img](/img/in-post/post-bg-salesforce-mule-18.png)

将 `Database` 的 `Insert` 操作拖出放入第一个选项框中.

![img](/img/in-post/post-bg-salesforce-mule-19.png)

设置 sql 语句, 和匹配的参数, 插入数据库.

![img](/img/in-post/post-bg-salesforce-mule-20.png)

启动 Mule Application:

![img](/img/in-post/post-bg-salesforce-mule-21.png)
![img](/img/in-post/post-bg-salesforce-mule-22.png)

在 Salesforce 中创建一条 Contact 记录, Lasrt Name 为: **MuleSoft CDC**

![img](/img/in-post/post-bg-salesforce-mule-23.png)

我们在 MuleSoft Application 里可以看到 CDC 事件已经触发, 并且 payload 日志已经打印出来 :

![img](/img/in-post/post-bg-salesforce-mule-24.png)

再来看下数据库更新情况, 有一条新的记录产生, sfdc_id 为刚才在 Salesforce 创建的联系人记录.

![img](/img/in-post/post-bg-salesforce-mule-25.png)




