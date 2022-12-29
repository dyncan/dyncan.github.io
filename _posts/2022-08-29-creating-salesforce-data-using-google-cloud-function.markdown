---
layout: post
title: "使用Google Cloud Function创建Salesforce数据"
subtitle: ""
date: 2022-08-29 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-gcp-salesforce.jpeg"
catalog: true
tags:
  - Salesforce 
  - Google Cloud
  - GCP
  - Cloud Function
---

由于之前有接触过 GCP 和 Salesforce 集成相关的项目, 但是没有认真的了解过中间的实现细节, 那么今天这篇文章主要介绍如何在Google Cloud上创建一个Cloud Function并认证连接到Salesforce.

在本篇内容中, 我们的挑战主要包括:

  - 创建一个 Google Cloud Function
  - 使用Secret Manager在GCP中存储Salesforce凭证
  - 使用Cloud Function与Salesforce进行认证,并创建 Salesforce 数据

**开始前准备:**

  - 注册 [Salesforce Developer Edition](https://developer.salesforce.com/signup)
  - 注册 [Google Free Tier](https://cloud.google.com/free/?utm_source=google&utm_medium=cpc&utm_campaign=na-US-all-en-dr-SKWS-all-all-trial-e-dr-1008076&utm_content=text-ad-none-any-DEV_c-CRE_265893083941-ADGP_Hybrid%20%7C%20AW%20SEM%20%7C%20BKWS%20%7C%20US%20%7C%20en%20%7C%20EXA%20~%20Gcp%20Free%20Tier-KWID_43700032443105337-kwd-413433661135&utm_term=KW_google%20free%20tier-ST_google%20free%20tier&gclid=CjwKCAjw8df2BRA3EiwAvfZWaHjg4K-Jhv625KDMkbKBtKS-3lPSbnLr1FGegR8wudCaOLnK-db9kxoCtY4QAvD_BwE)

> Tip: 为了可以使用 Google Cloud Function, 我们需要绑定一张自己的真实信用卡, 主要原因是Google为了校验操作的不是机器人, 过程不会扣任何费用.

简单的架构示意图:

![img](/img/in-post/post-gcp-salesforce.jpeg)


#### 步骤 1. Google Cloud Platform: 启用 Secret Manager

通过REST API访问Salesforce需要三部分;用户名,密码和Security Token. 在GCP内存储这类信息的最安全选择是 Secret Manager.它允许对这些凭证进行存储和版本控制, 以便安全使用. 下面介绍如何在Secret Manager里设置参数.

**步骤:**

  - 在 `google Console`界面, 选择 `Security`,
  - 在 `Security` 选择卡中选择 `Secret Manager`, 点击进入详情页

  ![img](/img/in-post/post-gcp-salesforce-secret-manager.png)

#### 步骤 2. Google Cloud Platform: 创建 Salesforce 凭证

**步骤:**

  - 在 `Secret Manager`, 点击 `CREATE SECRET`

  ![img](/img/in-post/post-gcp-salesforce-create-secret.png)

  ![img](/img/in-post/post-gcp-salesforce-create-secret-page.png)

  在 `CREATE SECRET` 页面: 
  - Name = **SF_USER_PROD**
  - Secret Value = `YOUR@SALESFORCEUSERNAME.com`
  - 点击 `Create Secret` 按钮
  - 对于 `SF_PASS_PROD` 和 `SF_TOKEN_PROD` 重复上面这些相同的步骤.
  
  ![img](/img/in-post/post-gcp-salesforce-secret-list.png)

#### 步骤 3. Google Cloud Platform: 创建 Google Cloud Function (Python)

现在, Salesforce 凭证已经安全存储, 接下来我们将创建一个云函数(Cloud Function). Cloud Function是Serverless,可移植的代码解决方案, 在运行时被调用. 在创建的这个云函数中, 由于谷歌没有与Salesforce原生集成,我们必须调用一个名为Simple Salesforce的第三方Python REST API库. 这使得我们可以通过REST API使用简化的方法来验证, 创建, 更新和查询Salesforce数据.

**步骤:**

  - 在 `google Console`界面, 选择 `Cloud Functions`,

  ![img](/img/in-post/post-gcp-salesforce-cloud-function.png)

  - 在 `Cloud Functions`界面, 点击 `CLOUD FUNCTION`,

  ![img](/img/in-post/post-gcp-salesforce-create-cloud-function.png)

  在 `CLOUD FUNCTION` 页面: 
  - Name = `salesforce-record-submission`
  - Memory Allocated = `128 MiB`
  - Trigger Type = `HTTP`
  - Source Code > `Inline editor`
  - Runtime = `Python 3.7`
  - 在 Main.py 编辑器中放以下内容.这是我们将从 HTTP Post 传入数据,进行 Salesforce 身份验证并将此服务中发布的数据作为新联系人提交到 Salesforce 的代码.

```python
from simple_salesforce import Salesforce, SalesforceLogin
from simple_salesforce import SFType
from google.cloud import secretmanager
import requests
import json
import os

def main(initial_request):

    sftype_object = os.environ["sftype_object"]

    try:

        request_json = initial_request.get_json()

        print("main - print: {}".format(request_json))

        #retrieve salesforce session and instance reference
        session_id, instance = sf_login()

        record = SFType(sftype_object,session_id,instance)

        #send payload to Salesforce API
        record.create(request_json)

        #parse response from Salesforce API
        record_submit = record.describe()

        print("main - record_submit: {}".format(record_submit))

        return "Main Request Passed"

    except Exception as error:

        print('Main Error: ' + repr(error))    


def sf_login():

    # Create the Secret Manager client.
    client = secretmanager.SecretManagerServiceClient()

    # Organize the Secret Keys
    sf_user_prod = "SF_USER_PROD"
    sf_pass_prod = "SF_PASS_PROD"
    sf_token_prod = "SF_TOKEN_PROD"
    
    # Pass in the GCP Project ID
    # This will be found on the Secret Manager > Secret > Secret Details
    # projects/[gcp_project_id]/secrets/[secret]
    project_id = os.environ["gcp_project_id"]
    
    # Obtain the Secret Name Path
    sf_user_prod_name = f"projects/{project_id}/secrets/{sf_user_prod}/versions/latest"
    sf_pass_prod_name = f"projects/{project_id}/secrets/{sf_pass_prod}/versions/latest"
    sf_token_prod_name = f"projects/{project_id}/secrets/{sf_token_prod}/versions/latest"   
    
    # Obtain the Latest Secret Version
    sf_user_prod_response = client.access_secret_version(sf_user_prod_name)
    sf_pass_prod_response = client.access_secret_version(sf_pass_prod_name)
    sf_token_prod_response = client.access_secret_version(sf_token_prod_name)

    # Parse the Secret Response & Decode Payload
    sf_user_prod_secret = sf_user_prod_response.payload.data.decode('UTF-8')  
    sf_pass_prod_secret = sf_pass_prod_response.payload.data.decode('UTF-8') 
    sf_token_prod_secret = sf_token_prod_response.payload.data.decode('UTF-8')     

    # Assign Variables to Pass into Salesforce Login
    sf_username = sf_user_prod_secret
    sf_password = sf_pass_prod_secret
    sf_token = sf_token_prod_secret

    try:

        # call salesforce Login
        # return Session ID and Instance
        session_id, instance = SalesforceLogin(
            username = sf_username,
            password = sf_password,
            security_token = sf_token)

        return session_id, instance

    except Exception as error:

        print('Login Error: ' + repr(error)) 
```
  - 在 Requirements.txt 编辑器中放以下内容. 这将为Python添加对Requests, Simple Salesforce和Google Secret Manager的依赖关系的引用.
  
  ```python
  # Function dependencies, for example:
  # package>=version
  requests>=2.20.0
  simple-salesforce>=0.74.2
  google-cloud-logging==1.11.0
  google-cloud-secret-manager==0.2.0
  ```
  - 点击 `Deploy` 等待部署结果.

  设置运行时变量:
  
  ![img](/img/in-post/post-gcp-salesforce-cloud-function-runtime-value.png)

#### 步骤 4. 测试

现在这个云功能已经创建完毕，现在可以测试创建一个新的联系人。在这个测试中，我们将模拟一个HTTP Post到这Cloud Function，使用包含联系人信息的标准JSON格式。

**步骤:**

 - 选中刚才创建的Cloud Function:

![img](/img/in-post/post-gcp-salesforce-cloud-function-list.png)

- 在Function 页面, 点击 `TESTING` tab
![img](/img/in-post/post-gcp-salesforce-cloud-function-test.png)

- 放入测试 JSON 数据, 点击`TESTING THE FUNCTION`:

![img](/img/in-post/post-gcp-salesforce-cloud-function-test-run.png)


查看日志:

  ![img](/img/in-post/post-gcp-salesforce-cloud-function-log-success.png)

查看Salesforce List View:

  ![img](/img/in-post/post-gcp-salesforce-contact-data.png)
