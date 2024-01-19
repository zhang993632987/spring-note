# 动态重新加载路由配置

动态重新加载路由的功能非常有用，因为它使我们能够在无需重新启动网关服务器的情况下更改路由映射。**现有路由可以迅速修改，而新路由则需要通过在环境中的每个网关服务器进行重新加载。**&#x20;

Spring Actuator 提供了一个基于 POST 的端点路由，即 **actuator/gateway/refresh**，调用此端点将触发重新加载路由配置。

> <mark style="color:orange;">**经过实验，调用 actuator/gateway/refresh 端点并未触发路由配置加载（怀疑该端点的有效性可能需要让配置文件与应用放在一块，即放在与 Gateway Server 的 application.yml 文件同路径下的其他的配置文件中）**</mark>
>
> <mark style="color:blue;">**但是，使用 @RefreshScope 和 actuator/refresh 确实可以刷新路由配置。**</mark>
>
> (官方文档：To **clear the routes cache**, make a POST request to /actuator/gateway/refresh. The request returns a 200 without a response body.)
