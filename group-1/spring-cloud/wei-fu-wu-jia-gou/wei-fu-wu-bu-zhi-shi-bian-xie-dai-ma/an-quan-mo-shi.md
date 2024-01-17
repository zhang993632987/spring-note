# 安全模式

下图展示了如何实现以下3种模式来构建可以保护微服务的验证服务：

<figure><img src="../../../../.gitbook/assets/image (5) (1) (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

### <mark style="color:blue;">**验证**</mark>

如何确定调用服务的服务客户端就是它们自己声称的那个。

### <mark style="color:blue;">**授权**</mark>

如何确定是否允许调用微服务的服务客户端执行它们正在尝试进行的操作。

### <mark style="color:blue;">**凭据管理和传播**</mark>

如何避免客户端要不断地提供它们的凭据信息才能访问事务中涉及的服务调用。

为实现这一点，我们需要了解如何使用基于令牌的安全标准，如**OAuth2**和**JSON Web Token（JWT）**，来获取可以从一个服务调用传递到另一个服务调用的令牌，以对用户进行验证和授权。
