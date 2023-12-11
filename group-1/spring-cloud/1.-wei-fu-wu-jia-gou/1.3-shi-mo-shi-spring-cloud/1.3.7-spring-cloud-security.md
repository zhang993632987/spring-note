# 1.3.7 Spring Cloud Security

<mark style="color:blue;">**Spring Cloud Security**</mark>是一个**验证**和**授权**框架，可以控制哪些人可以访问你的服务，以及他们可以用服务做什么。

因为Spring Cloud Security是**基于令牌**的，它允许服务通过验证服务器发出的令牌彼此进行通信。**接收HTTP调用的每个服务可以检查提供的令牌，以确认用户的身份以及用户对该服务的访问权限**。此外，Spring Cloud Security还支持**JSON Web Token（JWT）**。JWT框架标准化了创建OAuth2令牌的格式，并为创建的令牌进行数字签名提供了标准。
