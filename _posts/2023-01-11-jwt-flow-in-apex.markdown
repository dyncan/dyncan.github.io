---
layout: post
title: "在 Apex 中使用 JWT Flow"
subtitle: ""
date: 2023-01-11 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-jwt-apex.png"
catalog: true
tags:
  - Apex
  - JWT
  - Salesforce
---

在之前的[文章](https://dyncan.github.io/2023/01/08/salesforce-jwt-bearer-flow/)中介绍了什么是 JWT Token, 以及如何创建 JWT Token, 并通过 JWT Token 获取 Salesforce 访问令牌, 今天主要介绍如何在 Salesforce Apex 类中使用同样的功能. 目前 Salesforce 只支持Java Keystore(JKS)格式, 用于将私钥/公钥对(含证书)导入 Salesforce org 中.

### 将私钥/公钥对转换为 Salesforce Keystore JKS 文件

1. 在 server.key 文件的同一文件夹(通过之前的[文章](https://dyncan.github.io/2023/01/08/salesforce-jwt-bearer-flow/)创建的. 克隆server.key 文件并保存为 server.pem.
2. 执行如下命令: 

```
// 第一步
openssl pkcs12 -export -in server.crt -inkey server.pem -out keystore.p12

//第二步
keytool -importkeystore -srckeystore keystore.p12  -srcstoretype PKCS12 -destkeystore jwt_flow.jks -deststoretype JKS

//第三步
keytool -keystore jwt_flow.jks -changealias -alias 1 -destalias jwt_flow
```

### 将 JKS 文件导入 Salesforce 里

1. `setup`-> 搜索关键词 **Certificate and Key Management**
2. 点击 `Import From Keystore`.
3. 上传 `jwt_flow.jks` 文件, 并提供你创建 `jwt_flow.jks` 文件的密码.

### 在 Apex 里使用 JWT 

```java
public class JWTBearerFlow {
  public static String getAccessToken() {
    Auth.JWT jwt = new Auth.JWT();
    jwt.setSub('salesforce@example.com');
    jwt.setAud('https://<your instance>.salesforce.com');
    jwt.setIss('connected app client id');

    //Create the object that signs the JWT bearer token
    Auth.JWS jws = new Auth.JWS(jwt, 'Certificate keystore name');

    //Get the resulting JWS in case debugging is required
    String token = jws.getCompactSerialization();

    //Set the token endpoint that the JWT bearer token is posted to
    String tokenEndpoint = 'https://<your instance>.salesforce.com/services/oauth2/token';

    //POST the JWT bearer token
    Auth.JWTBearerTokenExchange bearer = new Auth.JWTBearerTokenExchange(
      tokenEndpoint,
      jws
    );

    //return access token
    return bearer.getAccessToken();
  }
}
```

不过我们发现上面代码的所有实现细节有被呈现出来, 这样的类是不安全的. 这在 Salesforce 中不是最佳实践. 我建议使用如下 `Named Credentials` 来实现:

如图所示:

![img](/img/in-post/post-bg-jwt-apex-01.png)

代码更新如下:

```java
String service_limits = '/services/data/v56.0/sobjects/Account/listviews/';

HttpRequest req = new HttpRequest();
req.setEndpoint('callout:JWT_Flow' + service_limits);
req.setMethod('GET');

Http http = new Http();
HTTPResponse res = http.send(req);
System.debug(res.getBody());
```



