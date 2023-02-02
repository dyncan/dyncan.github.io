---
layout: post
title: "DataWeave in Apex"
subtitle: ""
date: 2023-02-02 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-dataweave-in-apex.jpeg"
catalog: true
tags:
  - DataWeave
  - MuleSoft
---

在 Apex 中处理基于文本的数据是一项相当普遍的任务, 尤其是在系统集成中. 然而, 处理 JSON,XML和 CSV 往往需要写大量的代码, 并且也会导致性能问题, 目前 [DataWeave for Apex](https://help.salesforce.com/s/articleView?id=release-notes.rn_apex_DataWeaveInApex_DevPreview.htm&release=240&type=5)(截至2月3号,处于测试阶段) Salesforce 允许我们用 MuleSoft 的转换语言来处理文本数据. 只需两行代码, 就可以用 DataWeave 脚本处理所有常见的文本数据格式. 与 Apex 一样, DataWeave 脚本在 Salesforce 应用服务器内运行, 对执行的代码执行相同的堆(Heap)和 CPU 限制.

### DataWeave in Apex 是什么?

企业应用程序经常需要在 CSV, JSON, XML 与 Apex 对象等格式之间进行数据转换. Apex 中的 DataWeave 补充了 Apex 对 JSON 和 XML 处理的本地支持, 并使数据转换更容易编码, 更具可扩展性和效率. Apex 的开发人员可以更专注于解决业务问题, 而不是解决文件格式的具体问题. 而且你不需要成为 MuleSoft 的客户, 也不需要拥有任何特定的 Salesforce 许可证, 就可以在 Apex 中使用 DataWeave.

### 创建和部署一个简单的 DataWeave 项目

首先,我们需要创建一个 SFDX Project, 再连接一个 dev hub org, 之后在项目的 `force-app/main/default` 路径下创建一个文件夹 **dw**. 在这个文件夹里, 我们需要创建两个文件.

**json2sObjects.dwl**

```java
%dw 2.0
input incomingJson application/json
output application/json
---
incomingJson map {
    Company: $.title.rendered,
    FirstName: $.author
}
```
**json2sObjects.dwl-meta.xml**
```java
<?xml version="1.0" encoding="UTF-8"?>
<DataWeaveResource xmlns="http://soap.sforce.com/2006/04/metadata">
     <apiVersion>56.0</apiVersion>
</DataWeaveResource>
```

![img](/img/in-post/post-bg-DataWeave-in-apex-01.png)

代码创建完成之后, 从终端运行一个命令, 将代码推送到我们的scratch org中.

```
sfdx force:source:push --json --loglevel fatal
```

![img](/img/in-post/post-bg-DataWeave-in-apex-02.png)

#### 创建 Apex Class

文件创建完成之后, 我们需要创建一个 Apex Class, 这个class的作用是从一个公共的 REST API 中提取数据,将 response 响应传入一个 DataWeave 脚本, 并获取和序列化response. 确保能够将配置的 dwl 文件中的字段映射到指定的字段上去.

```java
public class DataWeaveInApex {

    private String incomingJson;
    
    public void consume(){
        Http http = new Http();
        HttpRequest request = new HttpRequest();
        request.setEndpoint('https://techcrunch.com/wp-json/wp/v2/posts?per_page=2');
        request.setMethod('GET');
        HttpResponse response = http.send(request);
        if(response.getStatusCode() == 200) {
            incomingJson = response.getBody();
            List<Lead> leads = (List<Lead>) translateToSObjects('Lead');
            System.debug('lead size: ' + leads.size());
            System.debug('lead results: ' + leads);
        }
    }

    public String translate() {
        Map<String, Object> parameters = new Map<String, Object>();
        parameters.put('incomingJson', incomingJson);
        DataWeave.Script script = DataWeave.Script.createScript(
            'json2sObjects'
        );
        DataWeave.Result result = script.execute(parameters);
        return result.getValueAsString();
    }

    public List<sObject> translateToSObjects(String sObjectType) {
        Type dynamicListType = Type.forName('List<' + sObjectType + '>');
        return (List<sObject>) JSON.deserialize(translate(), dynamicListType);
    }
}
```

![img](/img/in-post/post-bg-dataweave-in-apex-03.png)

### 在使用 DataWeave 的时候的一些考量或限制

- 目前每个 Org 支持最多 `50` 个 DataWeave 脚本.
- 在初始化 DataWeave 的时候会大量消耗 CPU 的时间
- Apex 类必须在 API 版本 `53.0` 或更高版本才能访问 DataWeave 集成方法。
- 一个 DataWeave 脚本的最大 body size 是 `100000` 个字符。
- DataWeave in Apex 不支持这些 content types : 
  - **Flat File Format (application/flatfile)**
  - **Excel (application/xlsx)** 
  - **Arvo (application/avro)**

