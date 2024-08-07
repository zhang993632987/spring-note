# 日志记录和跟踪模式

**微服务架构的缺点是调试、跟踪和监控问题要困难得多**，因为一个简单的操作可能会在应用程序中触发大量的微服务调用。出于这个原因，我们将研究以下 3 种核心日志记录和跟踪模式，以实现分布式跟踪：

### <mark style="color:blue;">**日志关联**</mark>

如何将一个用户事务的服务之间生成的所有日志联系在一起。

使用这种模式时，我们需要了解如何实现一个关联 ID，这是一个唯一的标识符，在一个事务中调用所有服务时都会携带它，它能够将每个服务生成的日志条目联系在一起。

### <mark style="color:blue;">**日志聚合**</mark>

使用这种模式，我们需要了解如何跨所有服务将微服务（及其各个实例）生成的所有日志合并到一个可查询的数据库中，并了解事务中涉及的服务的性能特征。

### <mark style="color:blue;">**微服务跟踪**</mark>

如何在涉及的所有服务中可视化客户端事务的流程，并了解事务所涉及的服务的性能特征。

<figure><img src="../../../../.gitbook/assets/image (6) (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>
