---
layout: post
title: "Salesforce 数据分析解决方案：Snowflake & Tableau"
subtitle: ""
date: 2023-02-04 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-salesforce-snowflake.jpeg"
catalog: true
tags:
  - Tableau CRM
  - Snowflake
  - Data Analysis
---

在 Salesforce 项目中，随着业务的不断增长，随之产生的数据量也越来越庞大，有的项目可能每年就会产生几千万甚至上亿条的数据，而这些数据随着时间推移逐渐变成历史数据，而这些数据也不想删除，因为历史数据可能还是要时常进行数据分析，产生各种报表，看看近几年的销售数据的走势。对于上亿条的数据，Salesforce 是没有办法保存的 ( 可以使用 Big Objects, 参考我之前的[文章](https://dyncan.github.io/2023/02/01/archive-salesforce-data-with-big-objects/) ), 更没有办法通过 Salesforce 报表来分析。接下来我会介绍两种工具或平台，主要作用是用于数据的归档和分析。

### Snowflake 是什么？

Snowflake 是一种计算和存储分离的云数据仓库一个数据仓库，它提供了一个灵活，可扩展和安全的解决方案，用于存储，处理和分析数据。它将传统的数据仓库与云计算技术结合起来，可以适应不断变化的数据需求.Snowflake 支持多种数据源和数据类型，可以在云中运行，为数据分析和数据工程师提供了更多的灵活性和可用性。

在 Snowflake 中，数据存储在云存储中，而计算则通过 Snowflake 的计算节点进行。这种分离计算和存储的架构使得 Snowflake 具有更高的灵活性和可伸缩性，可以适应不断变化的数据需求。此外，用户只需要支付实际使用的计算资源，而不必担心存储空间的限制。

#### Snowflake 的优势

- **云计算：** Snowflake 是一种云数据仓库，具有较高的可扩展性和可靠性。
- **计算和存储分离：** Snowflake 使用计算和存储分离的架构，使用者可以根据需要动态扩展计算和存储资源。
- **支持多种数据源：** Snowflake 支持多种数据源，包括结构化，半结构化和非结构化数据，使用者可以方便地管理不同类型的数据。
- **高效的数据分析：** Snowflake 提供了高效的数据分析功能，支持大规模数据的实时分析。
- **数据分享：** Snowflake 提供了方便的数据分享功能，可以安全地在多个组织间共享数据。

#### Snowflake 的劣势

- 运维成本较高：Snowflake 是一种云数据仓库，需要相对较高的运维成本。
- 安全性：Snowflake 存储的数据需要放在云端，需要考虑数据的安全性问题。
- 数据传输成本：由于 Snowflake 存储的数据在云端，数据的传输成本较高。
- 与现有系统的集成：Snowflake 是一种新兴的云数据仓库，与现有系统的集成可能有一定难度。

### Tableau 是什么

Salesforce 是在 2019 年 06 月 10 日宣布收购 Tableau. 它是一个专业的数据可视化软件，可以帮助用户快速创建和共享交互式的图形，以对数据进行分析和可视化。Tableau 可以连接到各种数据源 (比如 Snowflake), 并使用其强大的图形和图表功能帮助用户探索数据，挖掘有价值的洞察力。

#### Tableau 的优势

- 可视化功能：Tableau 拥有丰富的数据可视化功能，可以创建各种图表，帮助用户快速理解和分析数据。
- 易用性：它拥有易于使用的界面和拖放功能，使得非技术人员也能创建引人入胜的数据可视化。
- 数据连接：Tableau 可以连接多种数据源，包括 Excel, SQL Server, Oracle, Salesforce 等。
- 实时分析：Tableau 可以对实时数据进行分析，帮助用户及时了解数据的情况。
- 分享功能：Tableau 可以轻松分享分析结果和数据可视化，并可以在云端或其他平台上共享，以便多人协作分析数据。

#### Tableau 的劣势

- 高昂的价格：Tableau 的许多功能都以高昂的价格提供，这可能不适合那些预算有限的公司或个人
- 学习曲线较高：学习和使用 Tableau 可能需要一些时间和练习，特别是对于那些没有数据分析经验的人。
- 资源需求较高：使用 Tableau 可能需要更强的计算机硬件和内存，特别是当处理大量数据时。

### 与 Salesforce 集成

场景：将 Salesforce 的数据进行归档，同步至云数据仓库 Snowflake 中，然后通过 Tableau 连接 Snowflake 并作为数据源来展示 Salesforce 数据，最后进行数据分析。

我们来看下 Snowflake 官方给出的整体架构示意图：

![img](/img/in-post/post-bg-salesforce-snowflake-01.png)

#### 准备

- 注册 [CRM Analytics Account](https://trailhead.salesforce.com/promo/orgs/analytics-de)
- 注册 [Snowflake Account](https://signup.snowflake.com/)
- 下载 [Tableau 桌面版](https://www.tableau.com/products/desktop/download)
- 对 SQL 以及数据库概念有基本了解。

### Snowflake 环境的配置

Snowflake 账户注册完成之后，登陆到主页面，让我们先点击 `Worksheet` , Worksheet 的主要作用是在交互式环境中执行 SQL 命令和查询。它提供了一个简单的界面，使用户能够快速测试 SQL 语句，并实时显示查询结果。因此，Snowflake Worksheet 对于数据分析和快速查询数据非常有用。如图所示：

![img](/img/in-post/post-bg-salesforce-snowflake-03.png)

我们可以重新修改 Worksheet 的名字：

![img](/img/in-post/post-bg-salesforce-snowflake-04.png)

#### 创建 Snowflake 相关对象

因为我们需要将 Salesforce 数据同步到 Snowflake, 所以我们需要为此创建相关的 Role, User, Warehouse, Database, Scheme 等，这样做的好处是：我们可以更好的控制权限，每个流程都对应特定的用户和权限，方便管理，而不是都使用 Admin 的角色去做数据同步和集成。

将当前角色切换为 `SECURITYADMIN`:

> SECURITYADMIN 角色是 Snowflake 中的系统管理员角色。它拥有执行 Snowflake 安全管理任务的所有权限，包括创建，修改和删除角色，授予和撤销权限，管理用户，管理防火墙规则等。因此最好为少数受信任的用户分配该角色。我们会使用这个角色为我们的数据集成创建特定的角色和用户等。

![img](/img/in-post/post-bg-salesforce-snowflake-05.png)

在交互环境中执行如下 SQL 命令，以下 SQL 会创建一个新的角色叫 `CRM_ANALYST_ROLE`, 而且 SYSADMIN 角色将拥有 CRM_ANALYST_ROLE 角色的所有权限。

```sql
-- 创建角色 CRM_ANALYST_ROLE
CREATE OR REPLACE ROLE CRM_ANALYST_ROLE COMMENT='CRM ANALYST ROLE';
-- 将 CRM_ANALYST_ROLE 角色授权给 SYSADMIN
GRANT ROLE CRM_ANALYST_ROLE TO ROLE SYSADMIN;
```

执行以下 SQL 命令创建一个 User, 并将 CRM_ANALYST_ROLE 角色赋予用户 `CRM_ANALYST`. 请记住该用户名和密码，在之后 Salesforce 和 Tableau 集成中需要用到。

```sql
-- 创建用户和密码
CREATE OR REPLACE USER CRM_ANALYST PASSWORD='CRMSnowflake001' 
    DEFAULT_ROLE=CRM_ANALYST_ROLE 
    DEFAULT_WAREHOUSE=CRM_WH
    DEFAULT_NAMESPACE=CRM_DB.PUBLIC
    MUST_CHANGE_PASSWORD = FALSE
    COMMENT='CRM ANALYST';
GRANT ROLE CRM_ANALYST_ROLE TO USER CRM_ANALYST;
```

将当前角色切换为 `SYSADMIN`:

> **这里解释下为什么要切换角色：** 
> 
> **SYSADMIN** 角色具有对整个 Snowflake 实例的最高管理权限，可以执行任何操作，例如创建，修改或删除数据仓库，数据库，账户等.
> 
> **SECURITYADMIN** 角色具有管理 Snowflake 安全的权限，但不能执行所有操作，例如创建，修改或删除数据库。他们可以管理安全性相关的内容，例如创建和管理用户，角色，权限和安全策略.
> 
> 因此，SYSADMIN 角色拥有比 SECURITYADMIN 角色更多的权限。

可以通过 UI 切换角色，也可以通过以下 SQL 命令：

```sql
-- 切换 SYSADMIN 角色
USE ROLE SYSADMIN;
```

创建 Database 和 scheme, 将来 Salesforce 同步的数据将会存入到该数据库里

```sql
-- 创建数据库 CRM_DB
CREATE OR REPLACE DATABASE CRM_DB;
-- 创建 SCHEMA SALESFORCE
CREATE OR REPLACE SCHEMA CRM_DB.SALESFORCE;
```

授予 CRM_ANALYST_ROLE 在这些结构上的必要权限：

```sql
-- 将 CRM_DB 数据库的使用权限授予 CRM_ANALYST_ROLE 角色。这意味着 CRM_ANALYST_ROLE 角色现在可以连接到 CRM_DB 数据库并对其进行查询，但是不能执行任何写入操作
GRANT USAGE ON DATABASE CRM_DB TO ROLE CRM_ANALYST_ROLE;
-- 将 CRM_DB.SALESFORCE 的所有权限授予 CRM_ANALYST_ROLE 角色。这意味着 CRM_ANALYST_ROLE 角色现在可以对 CRM_DB.SALESFORCE 进行读取和写入操作，包括对数据表，字段，索引，视图等进行修改。
GRANT ALL ON SCHEMA CRM_DB.SALESFORCE TO ROLE CRM_ANALYST_ROLE;
```

接下来我们将创建一个 Snowflake WAREHOUSE, 我们还将在 WAREHOUSE 上授予 CRM_ANALYST_ROLE 使用权限：

> 在 Snowflake 中，WAREHOUSE 是一个计算集群，可以根据需要即时启动/停止或扩大/缩小。WAREHOUSE 只执行计算工作，而所有的数据都管理和存储在 DATABASES 中，因为 Snowflake 在存储和计算之间完全分离，允许快速和灵活的独立扩展。

```sql
CREATE OR REPLACE WAREHOUSE CRM_WH
      WITH WAREHOUSE_SIZE = 'XSMALL'
      AUTO_SUSPEND = 300
      AUTO_RESUME = TRUE;

GRANT USAGE ON WAREHOUSE CRM_WH TO ROLE CRM_ANALYST_ROLE;
```

### Salesforce 环境配置

在 Salesforce 中，点击 `Setup`, 在快速查找栏中，输入 `Analytics`.点击 `Settings`.

![img](/img/in-post/post-bg-salesforce-snowflake-06.png)

在 Salesforce 中，打开 Analytics Studio, 点击 `Data Manager`

![img](/img/in-post/post-bg-salesforce-snowflake-07.png)
![img](/img/in-post/post-bg-salesforce-snowflake-08.png)

在 Data Manager 界面，点击 `New Connection`: 

![img](/img/in-post/post-bg-salesforce-snowflake-09.png)

然后选择 `Output`, 选中 `Snowflake Output Connector`

![img](/img/in-post/post-bg-salesforce-snowflake-10.png)

然后根据之前在 Snowflake 中的配置，填入如下信息：

![img](/img/in-post/post-bg-salesforce-snowflake-11.png)

> **Note**: 通过查看登录到 Snowflake 用户界面的浏览器标签中的 URL, 可以找到 Snowflake 账户名称。复制 **https://** 之后和 **snowflakecomputing.com** 之前的字符，即 https://abcd123.us-east-1.snowflakecomputing.com, Account name 将是 `abcd123.us-east-1`.

配置 Connection Sync out:

![img](/img/in-post/post-bg-salesforce-snowflake-12.png)
![img](/img/in-post/post-bg-salesforce-snowflake-13.png)

然后选择需要同步的对象，比如 Account, 可以选择增量同步，或者全量同步。

![img](/img/in-post/post-bg-salesforce-snowflake-14.png)

同步完成之后，登陆 Snowflake, 执行如下 SQL 查询结果：

```sql
select * from  crm_db.salesforce.account
```

如图所示：

![img](/img/in-post/post-bg-salesforce-snowflake-15.png)

至此，我们已经成功将 Salesforce 的数据同步至 Snowflake.

### 配置 Tableau

打开 Tableau 软件，点击 `更多`

![img](/img/in-post/post-bg-salesforce-snowflake-16.png)

在搜索框中输入 `Snowflake` 并点击。

![img](/img/in-post/post-bg-salesforce-snowflake-17.png)

输入用户名密码等信息，该信息来自之前创建的 Snowflake 对象。

![img](/img/in-post/post-bg-salesforce-snowflake-18.png)

登陆成功之后，选择创建的数据库，scheme 和 table: 

![img](/img/in-post/post-bg-salesforce-snowflake-19.png)

打开新的工作表，我们可以选中需要查看的字段，至此 Salesforce 的 Account 数据就会展示在 Tableau 的页面上。

![img](/img/in-post/post-bg-salesforce-snowflake-20.png)



