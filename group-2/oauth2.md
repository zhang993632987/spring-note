# oauth2

在 Spring cloud 项目中集成 oauth2 遇到了不小的麻烦，主要在：

1. 配置与 keycloak 认证服务器的集成信息时
2. 实现 Authorization 的传播时

原因主要在于：

1. 对 oauth2 协议不够了解，对于 oauth2 协议的一些概念根本不清楚，导致阅读资料存在障碍。
2. 资料过于分散：spring cloud 、 spring boot 、spring security、keycloak 中都存在相关的信息，但都未统一，且很多资料只有示例没有说明。同时在 spring 上版本变化太过剧烈，且文档中的方案是deprecated。

解决办法：

1. 学习 spring security 的知识，最好也学习下 oauth2 协议的相关知识
2.  虽然资料分散，但可以找出其中的内在关联性：

    1. keycloak 是一个实现了 oauth2 协议的认证服务，那么 spring cloud 并并不需要直到认证服务的具体实现，一定可以单纯的使用 spring cloud oauth2 来直接进行集成
    2. spring cloud 中关于 oauth2 的部分较少，并建议读者阅读 springboot 关于 oauth2 的内容
    3. 在spring boot 给出的 migration 方案中，目标是将 oauth2 合并到 spring security 中去

    因此，最终应该参考 spring security 中的文档（关于Authorization 头的传递）和 springboot 中 oauth2 的配置
