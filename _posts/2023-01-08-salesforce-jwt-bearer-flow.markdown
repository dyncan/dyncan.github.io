---
layout: post
title: "Salesforce OAuth: JWT Bearer 流程"
subtitle: ""
date: 2023-01-08 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-salesforce-jwt-flow.png"
catalog: true
tags:
  - OAuth2.0
  - JWT
  - Salesforce
---

JWT Bearer Flow 是一种 OAuth 流程，其中外部应用程序 (也称为客户端) 向 Salesforce 发送一个称为 `JWT` 的签名 `JSON` 字符串，以获得一个`访问令牌(access token)`. 然后外部应用程序可以使用该访问令牌来读取和写入 Salesforce 中的数据。

与其他一些 OAuth 流程不同的是，JWT 流程不需要最终用户操作。外部应用程序发送 JWT 并进行自我认证，无需人工干预。

在本文开始之前，先来简单介绍一下 JWT 的构成，以此希望大家对整个 JWT Bearer Flow 的理解更加的清楚。

### JWT 的数据结构

真实的 JWT 大概像下面这样：

![img](/img/in-post/post-bg-salesforce-jwt-1.png)

JWT 是一个很长的字符串，中间用点`(.)`分隔成三个部分。需要注意的是：JWT 内部是没有换行的。

这三个部分组成如下：

  - Header(头部)
  - Payload(载荷)
  - Signature(签名)

#### Header

JWT 的头部有两部分信息：

  - 声明类型，JWT 令牌统一写为 JWT
  - 声明加密的算法，默认为 HMAC SHA256, 简写后 HS256

示例：

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

然后将头部通过 Base64URL 加密算法转成字符串，构成了第一部分。

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

#### Payload

Payload 部分也是一个 JSON 对象，用来存放实际需要传递的数据.JWT 规定了 7 个官方字段，但是这些字段不是必须的但是是推荐使用的。

  - iss (issuer):签发人
  - exp (expiration time):过期时间
  - sub (subject):主题
  - aud (audience):受众
  - nbf (Not Before):生效时间
  - iat (Issued At):签发时间
  - jti (JWT ID):编号

示例：

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
```

这部分的数据也是经过 Base64Url 进行加密的，这部分加密的内容组成了 JWT 的第二部分。

> 请注意：针对令牌这部分的签名已经被防范篡改。但是这部分还是可以被解密的，因此请不要将任何密钥放到这部分的数据中，除非你的密钥是已经加密过的密钥。

#### Signature

Signature 部分是对前两部分的签名，防止数据篡改。首先，需要指定一个密钥 (secret).这个密钥只有服务器才知道，不能泄露给用户。

例如，如果你希望使用 HMAC SHA256 算法来进行签名，那么这个算法中使用的数据为：

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

签名的作用主要用于校验传输的令牌 (Token) 数据没有在过程中被篡改。

算出签名以后，将上面的三个部分用`.`连接成一个完整的字符串，就构成了最终的 jwt.

## Salesforce OAuth JWT Flow

接下来让我们开始今天的内容。在这个例子中，我们将介绍如何使用 Salesforce 中的 JWT 流程获取 Access Token.

### 生成 private key 和 cert

作用：外部应用程序使用 private key 来签名 JWT, 而数字证书 (cert) 则由 Salesforce 用来验证签名并签发访问令牌。

使用 JWT Bearer Token flow 的第一步是创建证书。为了能够创建证书，你需要[安装 OpenSSL](https://github.com/openssl/openssl#download)

#### 步骤 1: 创建 RSA private key

运行以下 OpenSSL 命令：

```
openssl genrsa -des3 -passout pass:xxxx -out server.pass.key 2048
```

结果：
![img](/img/in-post/post-bg-salesforce-jwt-2.png)

#### 步骤 2: 从 server.pass.key 文件中创建 key file

运行以下 OpenSSL 命令：

```
openssl rsa -passin pass:xxxx -in server.pass.key -out server.key
rm server.pass.key (可选的, 可以删除这个文件)
```

结果：
![img](/img/in-post/post-bg-salesforce-jwt-3.png)


#### 步骤 3: 生成证书

运行以下 OpenSSL 命令：

```
openssl req -new -key server.key -out server.csr
```

结果：
![img](/img/in-post/post-bg-salesforce-jwt-4.png)

#### 步骤 4: 生成 SSL 证书

运行以下 OpenSSL 命令：

```
openssl x509 -req -sha256 -days 365 -in server.csr -signkey server.key -out server.crt
```

结果：
![img](/img/in-post/post-bg-salesforce-jwt-5.png)

### 在 Salesforce 中创建一个 Connected App

为了连接到我们的 Salesforce org，我们需要创建一个 Connected App.如果你是在 Lightning 或者 Classic 中，创建方式会略有不同。

#### 创建 connected app

  - Lightning: `Setup` ⇨ `App Manager`, 点击 `New Connected App`
  - Classic: `Setup` ⇨ Apps, 在 `Connected Apps` 列表选项中，点击 `New`.

  1. 填入 `Name` 和 `Email`
  2. 开启 `Enable OAuth Settings`
  3. Callback URL: `https://login.salesforce.com/services/oauth2/callback`
  4. 开启 `Use digital signatures`, 上传在上一步创建的数字证书 (server.crt)
  5. 选择 OAuth 范围：
     1. Manage user data via APIs (api)
     2. Manage user data via Web browsers (web)
     3. Perform requests at any time (refresh_token, offline_access)
  6. 点击 `Save`

#### 预先批准 connected app 

为什么需要这一步骤呢？先说结论，如果没有这一步，直接生成 JWT Token, 然后调用 Salesforce API 获取 Access Token 是会报错的，如下：

```json
 {"error":"invalid_grant","error_description":"user hasn't approved this consumer"}
```

从技术上讲，我们的应用程序发送的 JWT Token 进行认证，但是 salesforce 此时并不知道哪些资源是可用的，并得到了用户的许可。通常情况下，我们可以发送 `scope` 参数作为 JWT 的一部分去实现，但 Salesforce 不支持这样做。[参考文档](https://help.salesforce.com/articleView?id=remoteaccess_oauth_jwt_flow.htm&type=5#grants_access).

简单地说，这意味着在使用 jwt flow 之前至少获得一次刷新令牌 (下面的第二个选项)[选项 2], 或执行后端审批[下面的第一个选项].

这里提供了几个不同的选项可以做到这一点:
  - **选项 1:** 管理员从 Salesforce 的 connected app 中进行审批
  - **选项 2:** 用 `User-Agent OAuth Flow` 预先批准 connected app 

#### 选项 1

执行如下步骤：

  - `Setup` -> `App Manager` -> `YOUR APP` -> `Manage` -> `Edit Policies`
  - 在 `Permitted Users` 选择 `Admin approved users are pre-authorized`
  - 点击 `Save`.
  - 点击 `Manage Profiles`
  - 选择 `System Administrator`
  - 点击 `Save`

#### 选项 2

将下面链接中的参数替换为你自己的，然后复制链接到浏览器中：

```
https://<your instance>.salesforce.com/services/oauth2/authorize?response_type=token&client_id=<consumer key>&redirect_uri=https://login.salesforce.com/services/oauth2/callback
```

![img](/img/in-post/post-bg-salesforce-jwt-6.png)

### 创建 JWT Token

你可以使用你喜欢的开发语言的库来创建 JWT Token，但我这里只使用一个[在线工具](https://jwt.io/)来演示。

#### JWT Header

```json
{
  "alg": "RS256",
  "typ": "JWT"
}
```

#### JWT Payload

```json
{
  "iss": "Consumer key",
  "sub": "Username",
  "aud": "https://<login or test>.salesforce.com",
  "exp": "now + 2 minutes in Unix timestamp"
}
```

其中 `exp` 的时间，可以通过将下面 JS 复制到 chrome console 里：

```javascript
d = new Date(); 
d.setSeconds(d.getSeconds() + 180); // 有效期为 3 分钟
d.valueOf();
```

#### Signature

```
RSASHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  Private Key
)
```

从之前步骤中生成的 `Server.Key` 中复制 `Private Key`

### 通过 POSTMAN 使用 JWT 获取 Access Token

复制上一个步骤生成的 JWT Token:

![img](/img/in-post/post-bg-salesforce-jwt-8.png)

### 总结

JWT flow 在 Salesforce 生态系统中被广泛使用，以使外部应用程序能够访问 Salesforce 数据，而无需用户手动干预。通过向 Salesforce 发送签名的 jwt, 我们获得一个访问令牌 (Access Token). 上面的步骤应该适用于您的外部应用程序所使用的任何编程语言 (java,node,python 等).

