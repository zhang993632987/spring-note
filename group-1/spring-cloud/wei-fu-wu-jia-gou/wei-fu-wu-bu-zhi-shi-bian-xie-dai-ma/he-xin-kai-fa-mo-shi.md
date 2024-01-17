# 核心开发模式

<mark style="color:blue;">**核心开发模式**</mark>解决了**构建微服务的基础问题**，下图展示了构建微服务的基础：

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

### <mark style="color:blue;">**服务粒度**</mark>

如何将业务域分解为微服务，才能使每个微服务都具有适当程度的职责？

* 服务职责划分过于粗粒度，在不同的业务问题领域重叠，会使服务随着时间的推移变得难以维护。
* 服务职责划分过于细粒度，则会使应用程序的整体复杂性增加，并将服务变为无逻辑的“哑”数据抽象层。

### <mark style="color:blue;">**通信协议**</mark>

开发人员如何与服务进行通信？第一步是定义需要同步协议还是异步协议。

* 对于同步协议，最常见的协议是基于HTTP的REST，使用XML、JSON或诸如Thrift之类的二进制协议来与微服务来回传输数据。
* 对于异步协议，最流行的协议是高级消息队列协议（Advanced Message Queuing Protocol，AMQP），使用一对一（队列）或一对多（主题）的消息代理，如RabbitMQ、Apache Kafka和Amazon简单队列服务（Simple Queue Service，SQS）。

### <mark style="color:blue;">**接口设计**</mark>

如何设计实际的服务接口，便于开发人员进行服务调用？如何构建服务？最佳实践是什么？

### <mark style="color:blue;">**服务的配置管理**</mark>

如何管理微服务的配置，使其在不同的云环境之间移动？

### <mark style="color:blue;">**服务之间的事件处理**</mark>

如何使用事件解耦微服务，才能最小化服务之间的硬编码依赖关系，并提高应用程序的弹性？答案是<mark style="color:orange;">**使用Spring Cloud Stream的事件驱动架构**</mark>。
