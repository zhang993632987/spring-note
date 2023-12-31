# 隐式许可

**隐式（Implicit）许可通常用于完全在用户浏览器中运行的纯JavaScript应用程序**。在其他授权类型中，用户与执行用户请求的客户端服务器进行通信，客户端服务器与受保护的下游服务进行交互。而在隐式授权类型中，所有服务交互直接发生在用户的客户端（通常是一个Web浏览器）上。

<figure><img src="../../../../.gitbook/assets/file.excalidraw (4).svg" alt=""><figcaption></figcaption></figure>

使用**隐式（Implicit）许可类型**获取access token 的流程如下：

1. 在OAuth2授权服务上注册JavaScript应用程序。在注册时需要提供一个应用程序名称和一个回调URL，该URL将在用户的OAuth2访问令牌被重定向时使用。
2. JavaScript应用程序调用OAuth2服务。JavaScript应用程序必须提供一个预先注册的应用程序名称。然后，OAuth2服务器强制用户进行身份验证。
3.  如果用户成功进行身份验证，OAuth2服务不会返回一个令牌，而是将用户重定向回JavaScript应用程序所有者在步骤 1 中注册的页面。

    在重定向回用户的URL中，OAuth2访问令牌作为查询参数由OAuth2认证服务传递。
4. JavaScript应用程序接受传入请求并运行一个JavaScript脚本，解析OAuth2访问令牌并将其存储（通常作为Cookie）。
5. 每次调用受保护的资源时，OAuth2访问令牌都会被传递给被调用的服务。
6. 被调用的服务验证OAuth2令牌并检查用户是否被授权执行其尝试的活动。

> 关于OAuth2隐式许可类型，请记住以下几点：
>
> * 首先，隐式授权是唯一一种将OAuth2访问令牌直接暴露给公共客户端（Web浏览器）的授权类型。&#x20;
>   * 使用授权码许可类型时，客户端应用程序接收到OAuth2 授权服务器返回的授权码，客户端应用程序通过提交授权码来获取OAuth2令牌。返回的OAuth2令牌永远不会直接暴露给用户的浏览器。&#x20;
>   * 使用客户端凭证许可类型时，授权发生在两个基于服务器的应用程序之间；
>   * 而使用资源拥有者凭据许可类型时，请求服务的应用程序和服务都受到同一组织的信任和拥有。
> * 由隐式许可类型生成的OAuth2令牌更容易受到攻击和滥用，因为这些令牌对浏览器可见。
>   * 在浏览器中运行的任何恶意JavaScript都可以访问到OAuth2访问令牌并冒充用户调用服务。因此，隐式许可类型的OAuth2令牌应该是短暂的（1-2小时）。
>   * 由于OAuth2访问令牌存储在浏览器中，OAuth2规范不支持刷新令牌的概念，即无法自动续订令牌。
