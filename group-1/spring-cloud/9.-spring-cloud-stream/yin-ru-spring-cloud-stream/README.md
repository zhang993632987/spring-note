# 引入 Spring Cloud Stream

Spring Cloud Stream是用于构建基于消息驱动微服务应用程序的框架。它在Spring Boot的基础上构建独立的、生产级别的Spring应用程序，并利用Spring Integration提供与消息代理的连接。该框架提供了对多个中间件供应商的配置，引入了**持久的发布-订阅语义**、**消费者组**和**分区**的概念。

通过简单地将spring-cloud-stream的依赖项添加到应用程序的类路径中，即可立即连接到通过提供的spring-cloud-stream binder公开的消息代理，并可以使用简单的 **java.util.function.Function** 实现基于传入消息的功能需求。

## 应用模型

A Spring Cloud Stream application consists of a middleware-neutral core. The application communicates with the outside world by establishing _bindings_ between destinations exposed by the external brokers and input/output arguments in your code. Broker specific details necessary to establish bindings are handled by middleware-specific _Binder_ implementations.

<figure><img src="../../../../.gitbook/assets/SCSt-with-binder.png" alt=""><figcaption></figcaption></figure>

## 消费者群组

Spring Cloud Stream consumer groups are similar to and inspired by Kafka consumer groups.Each consumer binding can use the `spring.cloud.stream.bindings.<bindingName>.group` property to specify a group name.

All groups that subscribe to a given destination receive a copy of published data, but only one member of each group receives a given message from that destination. By default, when a group is not specified, Spring Cloud Stream assigns the application to an anonymous and independent single-member consumer group that is in a publish-subscribe relationship with all other consumer groups.

In general, it is preferable to always specify a consumer group when binding an application to a given destination. When scaling up a Spring Cloud Stream application, you must specify a consumer group for each of its input bindings. Doing so prevents the application’s instances from receiving duplicate messages (unless that behavior is desired, which is unusual).

## Consumer Types <a href="#consumer-types" id="consumer-types"></a>

Two types of consumer are supported:

* Message-driven (sometimes referred to as Asynchronous)
* Polled (sometimes referred to as Synchronous)

When you wish to control the rate at which messages are processed, you might want to use a synchronous consumer.

## 分区支持

Spring Cloud Stream provides support for partitioning data between multiple instances of a given application. In a partitioned scenario, the physical communication medium (such as the broker topic) is viewed as being structured into multiple partitions. One or more producer application instances send data to multiple consumer application instances and ensure that data identified by common characteristics are processed by the same consumer instance.

Spring Cloud Stream provides a common abstraction for implementing partitioned processing use cases in a uniform fashion. Partitioning can thus be used whether the broker itself is naturally partitioned (for example, Kafka) or not (for example, RabbitMQ).

> To set up a partitioned processing scenario, you must configure both the data-producing and the data-consuming ends.
