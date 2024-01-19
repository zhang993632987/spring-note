# 在Spring Cloud Gateway中配置路由

在其核心，<mark style="color:blue;">**Spring Cloud Gateway是一个反向代理**</mark>。

在微服务架构中，<mark style="color:blue;">**Spring Cloud Gateway 接收来自客户端的微服务调用，并将其转发到下游服务。**</mark>服务客户端认为它只与网关通信。

**为了与下游服务通信，网关必须知道如何将传入的调用映射到下游路由。**Spring Cloud Gateway有多种机制可以实现这一点，包括：

1. **使用服务发现进行自动路由映射：**通过服务发现，Spring Cloud Gateway可以自动映射路由，使其了解可用的下游服务及其位置，从而实现动态路由。
2. **使用服务发现进行手动路由映射：**除了自动映射之外，Spring Cloud Gateway还提供了手动映射路由的选项。这允许开发人员手动配置路由规则，指定特定的服务和路径映射。
