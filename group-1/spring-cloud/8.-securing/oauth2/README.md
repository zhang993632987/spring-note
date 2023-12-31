# OAuth2

<mark style="color:blue;">**OAuth2是一个基于令牌的安全框架，描述了授权授予的模式，但并未定义如何实际执行身份认证**</mark>。相反，它**允许用户使用第三方身份认证服务（称为身份提供者，IdP）进行身份认证**。如果用户成功进行身份认证，系统会生成一个令牌，用户必须将其与每个请求一同发送。然后，可以将令牌发回身份认证服务进行认证。

OAuth2的主要目标是，当多个服务被调用以满足用户的请求时，用户可以对每个服务进行身份认证，而无需向处理其请求的每个服务提供其凭据。OAuth2通过称为授权方式的身份认证方案来保护基于REST的服务。**OAuth2规范定义了四种授权方式：密码、客户端凭证、授权码、隐式。**

<mark style="color:blue;">**OAuth2的真正威力在于，它使应用程序开发人员能够轻松与第三方云提供商集成，并对用户进行身份认证和授权，而无需不断将用户的凭据传递给第三方服务。**</mark>

<mark style="color:blue;">**OpenID Connect（OIDC）**</mark>是建立在OAuth2框架之上的一层，**提供关于已登录应用程序用户（身份）的身份认证和个人资料信息。**当授权服务器支持OIDC时，有时将其称为**身份提供者**。

> 在 OAuth 2.0 的体系里面有 4 种角色：
>
> * 资源拥有者
> * 客户端（第三方软件）
> * 授权服务
> * 受保护资源

