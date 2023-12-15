# Spring Security

<mark style="color:blue;">**Spring Security**</mark> 是为基于 Spring 的应用程序提供声明式安全保护的安全性框架。Spring Security 提供了完整的安全性解决方案，它能够在 <mark style="color:blue;">**Web 请求级别**</mark>和<mark style="color:blue;">**方法调用级别**</mark>处理**身份认证**和**授权**。因为基于 Spring 框架，所以 Spring Security 充分利用了**依赖注入（dependency injection， DI）**和**面向切面（aspect-oriented programming，AOP）**的技术。

Spring Security 从两个角度来解决安全性问题：

* **它使用 Servlet 规范中的 Filter 保护 Web 请求并限制 URL 级别的访问**。
* Spring Security 还能够**使用 Spring AOP 保护方法调用** —— 借助于**对象代理**和**使用通知**，能够**确保只有具备适当权限的用户才能访问安全保护的方法**。
