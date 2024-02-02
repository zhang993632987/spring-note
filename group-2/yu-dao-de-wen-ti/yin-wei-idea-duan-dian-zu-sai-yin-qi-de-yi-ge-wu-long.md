# 因为IDEA断点阻塞引起的一个乌龙

## 问题描述

在测试 Spring Cloud Stream 中的消费者逻辑时，通过在消费者处理过程中打下断点，来跟踪消费者是否能够正常使用，主要为了测试 Consumer 的批量处理模式与 Kafka 的消息“累积”（当 Kafka 累积到一定的消息量后才响应 KafkaConsumer 的 poll 请求，相关配置为 <mark style="color:blue;">**fetch.min.bytes、fetch.max.wait.ms 和 max.poll.records**</mark>）。

采用的配置为：

```yaml
max.poll.records: 100     # 最多返回 100 条消息
fetch.min.bytes: 1048576  # 最少返回 1MB 的消息
fetch.max.wait.ms: 500    # 最多等待 500ms
```

为了测试批量功能是否正确，当向消费者订阅的主题发送消息时，连续多次调用发送消息的端口，但是往往在点击几次后，发送消息的流程便会阻塞，此时会触发消费者业务逻辑中的断点。

## 问题解决过程

遭遇问题是，我采用了无意识的快速点击发送消息端口。为了定位问题，所以我先是放慢了自身的点击速度，采用了先点击一次发送，然后再点击一次这样的方式。大概是因为 500ms 的时间过于巧合，在这种情况下，出现了【发送第一条消息不会出现问题，直到发送第二条消息才会出现问题的情况，也就是说只有在消费者处理完第一条消息后才允许第二条消息发送！】

### 排查 Spring Cloud Stream&#x20;

此时，我先是怀疑了是否是 Spring Cloud Stream 的封装逻辑是否进行过特殊的处理，但在仔细梳理 kafka 的逻辑后，却始终无法想到满足上述情形的设计方案，无法找出通过 kafka 做到【让生产者与消费者彼此配合，只有消费者处理完之前发送的消息后，生产者才能够继续发送消息】这样的逻辑。

所以，我只能是简单的修改了一下生产端的配置，其中主要的配置修改便是将 linger.ms 从 100 增大到了 10000，即从 100ms 增加到了 10s，也就是给与生产者 10s 的时间累积批次消息。

当进行了上述配置的修改后，先前观察到的【只有在消费者处理完第一条消息后才允许第二条消息发送！】完全不存在了，但是当消费者代码中的断点被触发时，生产者依然会阻塞。

此时，几乎可以排除先前的怀疑——【spring cloud stream 进行了特殊封装，其逻辑默认是同步的】。而为了进一步验证，采用了对生产者端打断点的方式，将断点打在了调用 send 方法之前的代码上（断点打在了 controller 方法的第一条代码上），然后发现当消费者完成消息消费后，生产者端的断点被触发，因而可以断定应用程序不是阻塞在 send 方法上了。

到此，几乎可以排除问题的来源在于 Spring Cloud Stream 了。

### 排查数据库连接池

通过检查控制台上的日志记录，偶然发现其中存在一条警告日志：

{% code overflow="wrap" %}
```log
2024-02-02T15:56:34.550+08:00  WARN 12756 --- [container-0-C-1] com.zaxxer.hikari.pool.PoolBase          : HikariPool-1 - Failed to validate connection com.mysql.cj.jdbc.ConnectionImpl@3d3cc5d2 (No operations allowed after connection closed.). Possibly consider using a shorter maxLifetime value.
```
{% endcode %}

上述警告似乎表明应用程序的数据库连接池不够用，因此将怀疑阻塞的原因是数据库连接不够用。

为此，通过配置调整了数据库连接池的大小，将连接池的大小从默认的 10 增大到了 100，但是在实验时问题依然存在。

### 排查线程池

受数据库连接池的启发，不得不将思路转移到线程池上面，因为不清楚 Spring Cloud Stream 的线程管理方案，因此怀疑是不是由于 Spring Cloud Stream 的消费者启用了占用了应用程序整体使用的线程，导致用于web访问线程不足，因此导致了阻塞。

为此，特意查询了 spring boot 应用内部使用的线程池方案，“百度”并未找到相关内容，以下内容来自于 chatgpt：

<details>

<summary><mark style="color:purple;">Spring Boot 项目中可能存在的线程池</mark></summary>

在一个Spring Boot项目中，可能会包含多个线程池，这些线程池用于执行不同的任务。以下是一些常见的Spring Boot项目中可能存在的线程池：

1. **主线程池（Main Thread Pool）**：这是应用程序启动时创建的主要线程池，用于处理主要的业务逻辑。这通常是由Spring Boot自动配置的线程池，例如`TaskExecutor`。
2. **异步方法线程池（Async Method Thread Pool）**：Spring Boot允许通过`@Async`注解实现异步方法，这些异步方法将在单独的线程中执行。默认情况下，Spring Boot会创建一个简单的线程池用于处理这些异步方法。
3. **调度线程池（Scheduling Thread Pool）**：如果在应用程序中使用了Spring的定时任务（`@Scheduled`注解），则会有一个用于执行定时任务的线程池。
4. **自定义线程池（Custom Thread Pools）**：在某些情况下，开发人员可能会自定义线程池来处理特定的任务或异步操作。
5. **连接池（Connection Pool）**：虽然不是线程池，但在数据库访问方面，连接池也是一个常见的概念。Spring Boot应用程序通常会使用连接池来管理数据库连接，以提高性能。
6. **Web服务器线程池（Web Server Thread Pool）**：如果你的Spring Boot应用程序是一个Web应用程序，那么底层的Web服务器（如Tomcat、Jetty或Undertow）也可能有自己的线程池用于处理HTTP请求。

</details>

经过理性分析，觉得上述怀疑似乎不太合理。为了彻底排除上述怀疑，查询了 spring boot 日志格式，观察各个方法的调用线程信息。

**默认情况下，从左到右，spring boot 的日志中各字段的含义如下：**

* **日志打印时间**
* **日志级别**
* **线程 ID**
* **线程名称**
* **执行的类名**
* **日志信息**

消费者中的日志信息如下：

{% code overflow="wrap" %}
```log
2024-02-02T15:34:48.284+08:00  INFO 12756 --- [container-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-miaosha-order-5, groupId=miaosha-order] Adding newly assigned partitions: 
```
{% endcode %}

而 controller 的日志信息如下：

{% code overflow="wrap" %}
```log
2024-02-02T15:57:37.352+08:00 DEBUG 12756 --- [nio-8080-exec-6] c.s.m.mapper.MiaoshaGoodsMapper.insert   : <==    Updates: 1
```
{% endcode %}

对比两者，可以发现，两者不可能处于同一个线程池中。而且观察日志，可以发现每次消费者日志的【线程ID】和【线程名称】都一样，因此可以猜测 spring cloud stream 使用一个额外的线程处理消费逻辑。

### IDEA断点

所有能够想到的原因都被排除后，问题依然没有得到解决，最后只能去怀疑是不是断点机制的问题。

为了验证这个猜想，去掉了消费者中的断点，而是将断点打在一个 controller 中的方法上，然后发现，不仅断点触发的方法被阻塞，调用其他controller 接口的方法也被阻塞。

然后，百度了以下 ide 的断点阻塞机制，发现确实存在【触发断点时，会暂停整个应用程序】这样的机制。
