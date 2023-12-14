# 使用用户进行认证

## 查询令牌申领端点

让我们启动我们的身份验证服务。为此，请在左侧菜单中点击“**Realm Settings**”选项，然后点击“**OpenID Endpoint Configuration**”链接，以查看领域可用端点的列表。

<figure><img src="../../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

<details>

<summary><mark style="color:purple;">http://192.168.10.110:8073/realms/learning-realm/.well-known/openid-configuration</mark></summary>

{% code overflow="wrap" %}
```json
{
    "issuer": "http://192.168.10.110:8073/realms/learning-realm",
    "authorization_endpoint": "http://192.168.10.110:8073/realms/learning-realm/protocol/openid-connect/auth",
    "token_endpoint": "http://192.168.10.110:8073/realms/learning-realm/protocol/openid-connect/token",
    "introspection_endpoint": "http://192.168.10.110:8073/realms/learning-realm/protocol/openid-connect/token/introspect",
    "userinfo_endpoint": "http://192.168.10.110:8073/realms/learning-realm/protocol/openid-connect/userinfo",
    "end_session_endpoint": "http://192.168.10.110:8073/realms/learning-realm/protocol/openid-connect/logout",
    "frontchannel_logout_session_supported": true,
    "frontchannel_logout_supported": true,
    "jwks_uri": "http://192.168.10.110:8073/realms/learning-realm/protocol/openid-connect/certs",
    "check_session_iframe": "http://192.168.10.110:8073/realms/learning-realm/protocol/openid-connect/login-status-iframe.html",
    ...
}
```
{% endcode %}

</details>

## 获取令牌

<mark style="color:blue;">**JSON 响应中的 token\_endpoint 是令牌的申领端点。**</mark>

使用Postman向端点 http://192.168.10.110:8073/realms/learning-realm/protocol/openid-connect/token 发送POST请求，然后提供**应用程序**、**密钥**、**用户ID**和**密码**。使用**基本身份验证**将这些元素传递到身份验证服务器端点。

下图显示了如何设置Postman以执行基本身份验证调用。请注意，我们将使用之前定义的**应用程序名称**和**应用程序密钥**作为密码：

<figure><img src="../../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

一旦配置了应用程序名称和密钥，我们需要将以下信息作为HTTP表单参数传递给服务：

* **grant\_type**：要执行的授权类型。在这个例子中，我们将使用 password 授权。
* **username**：登录用户的名称。
* **password**：登录用户的密码。

下图显示了我们的身份验证调用配置这些HTTP表单参数的方式。

<figure><img src="../../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

<details>

<summary><strong>JSON格式的响应</strong></summary>

```json
{
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJMZFZMdFFXODNxZkp5aGZOLUdJejBwazdLeGx0T3BCckdYQjJrVFhSS21JIn0.eyJleHAiOjE3MDI0NjE3NTEsImlhdCI6MTcwMjQ2MTQ1MSwianRpIjoiMGRiYTRjNDktZDdlMi00Y2Y1LTgzYjctZmNiOTk2NTg5MWVhIiwiaXNzIjoiaHR0cDovLzE5Mi4xNjguMTAuMTEwOjgwNzMvcmVhbG1zL2xlYXJuaW5nLXJlYWxtIiwiYXVkIjoiYWNjb3VudCIsInN1YiI6IjA3YWQwYjBhLTJjYzItNDMwNS04MDc0LWJiYjhkYTA5NDFjNiIsInR5cCI6IkJlYXJlciIsImF6cCI6ImxlYXJuaW5nIiwic2Vzc2lvbl9zdGF0ZSI6Ijk4NTBhOWI2LTk0NjktNDEwZi04YzQ1LWU3ZTVhYzY2MWVhMCIsImFjciI6IjEiLCJhbGxvd2VkLW9yaWdpbnMiOlsiLyoiXSwicmVhbG1fYWNjZXNzIjp7InJvbGVzIjpbImRlZmF1bHQtcm9sZXMtbGVhcm5pbmctcmVhbG0iLCJvZmZsaW5lX2FjY2VzcyIsInVtYV9hdXRob3JpemF0aW9uIiwibGVhcm5pbmctYWRtaW4iXX0sInJlc291cmNlX2FjY2VzcyI6eyJsZWFybmluZyI6eyJyb2xlcyI6WyJBRE1JTiJdfSwiYWNjb3VudCI6eyJyb2xlcyI6WyJtYW5hZ2UtYWNjb3VudCIsIm1hbmFnZS1hY2NvdW50LWxpbmtzIiwidmlldy1wcm9maWxlIl19fSwic2NvcGUiOiJwcm9maWxlIGVtYWlsIiwic2lkIjoiOTg1MGE5YjYtOTQ2OS00MTBmLThjNDUtZTdlNWFjNjYxZWEwIiwiZW1haWxfdmVyaWZpZWQiOmZhbHNlLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJhZG1pbiJ9.AX1VB9fEDVb_EZPJc6T3LIji6W1RXryqd4rzL7-tXFcVVQ4rxKJiV4V6RFn0YSHsiOAQOdgswXyVy1fhc6ScFGO9uNvkWcPcIo4vJilTXu26N2Vz_HPoMlxygJ4Ic0JVG3nUKYjc7EcPym6NSyjoCiYrgMI2lAPopUP_zFyaSZs3EoXRYuc3CUZlyvwaUvthD44CmSyqg-xTDd3OwVCmizd6n3l1O6S1ZwzHZslFlIqzcdVu4RPBHBeMAYrCyTq2IZ1ofCk1uo004pvCZazXwB2Crd5M5RjFvGRtTnaQw6jrtq3rsBzS6WH9iH97hL-994nwg2VcLdY38Xgi2etiWQ",
    "expires_in": 300,
    "refresh_expires_in": 1800,
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJjMzhmOGMxOC1jMjBlLTRlYmEtYTgwYS1mMWMyN2I3ZDhhNjAifQ.eyJleHAiOjE3MDI0NjMyNTEsImlhdCI6MTcwMjQ2MTQ1MSwianRpIjoiYzIyNjllMDAtZDdhZi00ODZhLWI5ZDYtM2I0YWI4ZTNlNDBmIiwiaXNzIjoiaHR0cDovLzE5Mi4xNjguMTAuMTEwOjgwNzMvcmVhbG1zL2xlYXJuaW5nLXJlYWxtIiwiYXVkIjoiaHR0cDovLzE5Mi4xNjguMTAuMTEwOjgwNzMvcmVhbG1zL2xlYXJuaW5nLXJlYWxtIiwic3ViIjoiMDdhZDBiMGEtMmNjMi00MzA1LTgwNzQtYmJiOGRhMDk0MWM2IiwidHlwIjoiUmVmcmVzaCIsImF6cCI6ImxlYXJuaW5nIiwic2Vzc2lvbl9zdGF0ZSI6Ijk4NTBhOWI2LTk0NjktNDEwZi04YzQ1LWU3ZTVhYzY2MWVhMCIsInNjb3BlIjoicHJvZmlsZSBlbWFpbCIsInNpZCI6Ijk4NTBhOWI2LTk0NjktNDEwZi04YzQ1LWU3ZTVhYzY2MWVhMCJ9.ShGZ8m4sOxMECdErPrsqx1K6yP2UfKU1BCzWBaXPzLM",
    "token_type": "Bearer",
    "not-before-policy": 0,
    "session_state": "9850a9b6-9469-410f-8c45-e7e5ac661ea0",
    "scope": "profile email"
}
```

</details>

* **access\_token**：用户对受保护资源进行的调用时呈现的访问令牌。
* **token\_type**：授权规范允许我们定义多种令牌类型，其中最常见的是Bearer Token（持有令牌）。
* **refresh\_token**：可以在令牌过期后呈现回授权服务器以重新发行令牌。
* **expires\_in**：访问令牌过期之前的秒数。
* **scope**：定义此访问令牌有效的范围。

现在我们从授权服务器检索到了有效的访问令牌，我们可以使用 [https://jwt.io](https://jwt.io/) 对JWT进行解码，以获取所有访问令牌的信息。
