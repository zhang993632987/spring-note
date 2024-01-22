# Spring Cloud Stream 使用范例: 分布式缓存

licensing 服务将总是检查分布式 Redis 缓存，以获取与特定许可相关联的组织数据。

* 如果组织数据存在于缓存中，将从缓存中返回数据。
* 如果不存在，则调用 organization 服务，并将调用结果缓存到 Redis 哈希中。&#x20;

当 organization 服务中的数据更新时，organization 服务将向 Kafka 发送一条消息。licensing 服务将捕获该消息并向 Redis 发送 DELETE 请求，以清除缓存。
