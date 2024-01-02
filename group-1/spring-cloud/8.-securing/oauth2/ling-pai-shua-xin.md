# 令牌刷新

<figure><img src="../../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

客户端可以向OAuth2授权服务提交**刷新令牌**，服务将验证刷新令牌，然后颁发一个新的OAuth2访问令牌，其流程如下：

* 用户登录到O-stock，但已经在OAuth2服务上进行了身份验证。用户愉快地工作，但不幸的是，他们的令牌过期了。&#x20;
* 用户下次尝试调用一个服务（比如组织服务）时，O-stock应用程序将过期的令牌传递给组织服务。 组织服务尝试使用OAuth2服务验证令牌，OAuth2服务返回HTTP状态码401（未授权）和一个JSON负载，指示令牌不再有效。组织服务将HTTP 401状态码返回给调用服务。&#x20;
* O-stock应用程序获取来自组织服务的401 HTTP状态码和JSON负载。然后，O-stock应用程序使用刷新令牌调用OAuth2认证服务。OAuth2认证服务验证刷新令牌，然后发送一个新的访问令牌。
