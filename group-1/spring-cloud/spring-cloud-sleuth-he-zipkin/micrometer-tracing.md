# Micrometer Tracing

> Spring团队意识到追踪功能可以从Spring Cloud中分离出来，并创建了**Micrometer Tracing**项目，本质上是Spring无关的Spring Cloud Sleuth的复制品。&#x20;
>
> **Spring Boot Actuator提供了对Micrometer Tracing的依赖管理和自动配置。**

## 添加依赖

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>
```

要将当前的 trace ID 和 span ID包含在日志中，可以通过将 **logging.pattern.level** 属性设置为 **%5p \[${spring.application.name:},%X{traceId:-},%X{spanId:-}]** 来实现。

```yaml
logging:
  pattern:
    level: '%5p [${spring.application.name:},%X{traceId:-},%X{spanId:-}]'
```

{% hint style="info" %}
## <mark style="color:blue;">提示</mark>

<mark style="color:blue;">**为了在网络上自动传播追踪信息，请使用自动配置的 RestTemplateBuilder 或 WebClient.Builder 来构建客户端。**</mark>

如果在不使用自动配置的构建器创建 WebClient 或 RestTemplate，自动追踪传播将无法正常工作！
{% endhint %}

{% hint style="info" %}
## <mark style="color:blue;">提示</mark>

If all of the following conditions are true, a **MicrometerObservationCapability** bean is created and registered so that your Feign client is observable by Micrometer:

* feign-micrometer is on the classpath
* A ObservationRegistry bean is available
* feign micrometer properties are set to true (by default)
  * spring.cloud.openfeign.micrometer.enabled=true (for all clients)
  * spring.cloud.openfeign.client.config.feignName.micrometer.enabled=true (for a single client)

If your application already uses Micrometer, enabling this feature is as simple as putting feign-micrometer onto your classpath:

如果以下所有条件都为真，则会创建并注册一个 **MicrometerObservationCapability** bean，以便通过 Micrometer 观察到 Feign 客户端：

* **feign-micrometer 在类路径上**
* **存在 ObservationRegistry bean**
* feign micrometer 属性设置为 true（默认情况下）
  * **spring.cloud.openfeign.micrometer.enabled=true**（对于所有客户端）
  * **spring.cloud.openfeign.client.config.feignName.micrometer.enabled=true**（对于单个客户端）

如果你的应用程序已经使用了 Micrometer，只需要添加以下依赖即可：

```xml
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-micrometer</artifactId>
</dependency>
```
{% endhint %}

## 追踪日志的结构

如果一切都设置正确，**服务中的日志语句便会包含追踪信息**。例如，如果对 organization 服务发出HTTP GET请求，将产生以下日志记录：

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Spring Cloud Sleuth在每个日志条目中添加了三个信息片段：

1. 应用程序名称。
2. Trace ID，相当于关联ID。这是一个唯一的数字，代表整个事务。
3. Span ID，是代表整个事务的一部分的唯一标识符。参与事务的每个服务都将有自己的span ID。
