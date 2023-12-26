# 配置选项

## Binding Service Properties <a href="#_binding_service_properties" id="_binding_service_properties"></a>

这些属性通过 **org.springframework.cloud.stream.config.BindingServiceProperties** 暴露出来：

* **spring.cloud.stream.instanceCount**：应用程序的部署实例数。**在生产者一侧进行分区时必须设置。**在使用 RabbitMQ 以及 Kafka（如果 autoRebalanceEnabled=false）的消费者一侧也必须设置。默认值：1。
* **spring.cloud.stream.instanceIndex**：应用程序的实例索引，从 0 到 instanceCount - 1 的数字。在使用 RabbitMQ 以及 Kafka（如果 autoRebalanceEnabled=false）进行分区时使用。在 Cloud Foundry 中会自动设置，以匹配应用程序的实例索引。
* **spring.cloud.stream.dynamicDestinations**：可以动态绑定的目的地列表（例如，在动态路由场景中）。**如果设置，只能绑定列出的目的地**。默认值：**空（允许绑定任何目的地**）。
* **spring.cloud.stream.defaultBinder**：如果配置了多个绑定器，则要**使用的默认绑定器**。默认值：空。
* **spring.cloud.stream.overrideCloudConnectors**：仅在激活云配置文件且应用程序提供 Spring Cloud Connectors 时适用。如果属性值为 false（默认值），则绑定器检测适用的已绑定服务（例如，Cloud Foundry 中为 RabbitMQ 绑定的服务）并使用它来创建连接（通常通过 Spring Cloud Connectors）。当设置为 true 时，此属性指示绑定器完全忽略已绑定的服务，依赖于 Spring Boot 属性（例如，依赖于为 RabbitMQ 绑定器提供的环境中的 spring.rabbitmq.\* 属性）。此属性的典型用法是在连接到多个系统时嵌套在自定义环境中。
* **spring.cloud.stream.bindingRetryInterval**：在**重试绑定创建时的间隔**（以秒为单位），例如，当绑定器不支持迟绑定且代理服务器（例如，Apache Kafka）宕机时。将其设置为零以将这些条件视为致命错误，阻止应用程序启动。默认值：30。

## Binding Properties <a href="#binding-properties" id="binding-properties"></a>

绑定属性通过以下格式提供：**spring.cloud.stream.bindings.\<bindingName>.\<property>=\<value>**。其中 \<bindingName> 表示正在配置的绑定的名称。

为了避免重复，Spring Cloud Stream 支持设置所有绑定的默认值，格式为 **spring.cloud.stream.default.\<property>=\<value>**，以及 **spring.cloud.stream.default.\<producer|consumer>.\<property>=\<value>** 。

### 通用绑定属性

这些属性通过 **org.springframework.cloud.stream.config.BindingProperties** 暴露出来，以下是适用于输入和输出绑定的绑定属性，必须以 **spring.cloud.stream.bindings.\<bindingName>** 为前缀：

* **destination**：绑定在中间件上的目的地（例如，RabbitMQ exchange 或 Kafka topic）。如果绑定表示消费者绑定（输入），则可以将其绑定到多个目的地，为以逗号进行分隔。此属性的默认值不能被覆盖。
* **group**：绑定的消费者组。仅适用于入站绑定。默认值：null（表示匿名消费者）。
* **contentType**：此绑定的内容类型。默认值：application/json。
* **binder**：此绑定使用的绑定器。默认值：null（如果存在默认绑定器，则使用它）。

> 可以使用 **spring.cloud.stream.default** 前缀设置默认值。

### **消费者属性**

这些属性通过 **org.springframework.cloud.stream.binder.ConsumerProperties** 暴露出来。以下是仅适用于输入绑定的绑定属性，必须以 **spring.cloud.stream.bindings.\<bindingName>.consumer** 为前缀：

* **autoStartup**：指示是否自动启动此消费者。默认值：true。
* **concurrency**：入站消费者的并发性。默认值：1。
* **partitioned**：消费者是否从分区生产者接收数据。默认值：false。
* **headerMode**：当设置为 none 时，在输入上禁用标头解析。仅对原生不支持消息标头且需要嵌入标头的消息中间件有效。当从不支持本机标头的非 Spring Cloud Stream 应用程序消费数据时，此选项很有用。当设置为 headers 时，它使用中间件的本机标头机制。当设置为 embeddedHeaders 时，它将标头嵌入到消息有效载荷中。默认值：取决于绑定器实现。
* `maxAttempts`：处理消息的尝试次数（包括第一次）。设置为 1 以禁用重试。默认值：3。
* `backOffInitialInterval`：初始重试间隔。默认值：1000。
* `backOffMaxInterval`：最大重试间隔。默认值：10000。
* `backOffMultiplier`：重试间隔的倍增因子。 默认值：2.0。
* `defaultRetryable`：监听器抛出的未在 `retryableExceptions` 中列出的异常是否可重试。默认值：true。
* `instanceCount`：当设置为大于等于零的值时，允许自定义此消费者的实例数（如果与 `spring.cloud.stream.instanceCount` 不同）。当设置为负值时，默认为 `spring.cloud.stream.instanceCount`。默认值：-1。
* `instanceIndex`：当设置为大于等于零的值时，允许自定义此消费者的实例索引（如果与 `spring.cloud.stream.instanceIndex` 不同）。当设置为负值时，默认为 `spring.cloud.stream.instanceIndex`。如果提供了 `instanceIndexList`，则忽略。默认值：-1。
* `instanceIndexList`：与不支持本机分区的绑定器一起使用（例如 RabbitMQ）；允许应用程序实例从多个分区中消耗。默认值：空。
* `retryableExceptions`：Throwable 类名的映射，键是类名，值是布尔值。指定将或不将重试的这些异常（和子类）。还参见 `defaultRetriable`。示例：`spring.cloud.stream.bindings.input.consumer.retryable-exceptions.java.lang.IllegalStateException=false`。默认值：空。
* `useNativeDecoding`：当设置为 true 时，入站消息将由客户端库直接反序列化，必须相应地进行配置（例如，设置适当的 Kafka 生产者值反序列化器）。当使用此配置时，入站消息的解组不基于绑定的 contentType。使用本机编码和解码时，生产者负责使用适当的编码器（例如，Kafka 生产者值序列化器）对出站消息进行序列化。此外，当使用本机编码和解码时，忽略 `headerMode=embeddedHeaders` 属性，标头不嵌入到消息中。参见生产者属性 `useNativeEncoding`。默认值：false。
* `multiplex`：当设置为 true 时，底层绑定器将在相同的输入绑定上本地多路复用目的地。默认值：false。

> **可以使用 spring.cloud.stream.default.consumer 前缀设置默认值。**

Default values can be set by using the `spring.cloud.stream.default.consumer` prefix.

* autoStartup
  * Signals if this consumer needs to be started automatically
  * Default: `true`.
* concurrency
  * The concurrency of the inbound consumer.
  * Default: `1`.
* partitioned
  * Whether the consumer receives data from a partitioned producer.
  * Default: `false`.
* headerMode
  * When set to `none`, disables header parsing on input. Effective only for messaging middleware that does not support message headers natively and requires header embedding. This option is useful when consuming data from non-Spring Cloud Stream applications when native headers are not supported. When set to `headers`, it uses the middleware’s native header mechanism. When set to `embeddedHeaders`, it embeds headers into the message payload.
  * Default: depends on the binder implementation.
* maxAttempts
  * If processing fails, the number of attempts to process the message (including the first). Set to `1` to disable retry.
  * Default: `3`.
* backOffInitialInterval
  * The backoff initial interval on retry.
  * Default: `1000`.
* backOffMaxInterval
  * The maximum backoff interval.
  * Default: `10000`.
* backOffMultiplier
  * The backoff multiplier.
  * Default: `2.0`.
* defaultRetryable
  * Whether exceptions thrown by the listener that are not listed in the `retryableExceptions` are retryable.
  * Default: `true`.
* instanceCount
  * When set to a value greater than equal to zero, it allows customizing the instance count of this consumer (if different from `spring.cloud.stream.instanceCount`). When set to a negative value, it defaults to `spring.cloud.stream.instanceCount`.&#x20;
  * Default: `-1`.
* instanceIndex
  * When set to a value greater than equal to zero, it allows customizing the instance index of this consumer (if different from `spring.cloud.stream.instanceIndex`). When set to a negative value, it defaults to `spring.cloud.stream.instanceIndex`. Ignored if `instanceIndexList` is provided.
  * Default: `-1`.
* instanceIndexList
  * Used with binders that do not support native partitioning (such as RabbitMQ); allows an application instance to consume from more than one partition.
  * Default: empty.
* retryableExceptions
  * A map of Throwable class names in the key and a boolean in the value. Specify those exceptions (and subclasses) that will or won’t be retried. Also see `defaultRetriable`. Example: `spring.cloud.stream.bindings.input.consumer.retryable-exceptions.java.lang.IllegalStateException=false`.
  * Default: empty.
* useNativeDecoding
  * When set to `true`, the inbound message is deserialized directly by the client library, which must be configured correspondingly (for example, setting an appropriate Kafka producer value deserializer). When this configuration is being used, the inbound message unmarshalling is not based on the `contentType` of the binding. When native decoding is used, it is the responsibility of the producer to use an appropriate encoder (for example, the Kafka producer value serializer) to serialize the outbound message. Also, when native encoding and decoding is used, the `headerMode=embeddedHeaders` property is ignored and headers are not embedded in the message. See the producer property `useNativeEncoding`.
  * Default: `false`.
* multiplex
  * When set to true, the underlying binder will natively multiplex destinations on the same input binding.
  * Default: `false`.

### 生产者属性

These properties are exposed via `org.springframework.cloud.stream.binder.ProducerProperties.`

The following binding properties are available for output bindings only and must be prefixed with `spring.cloud.stream.bindings.<bindingName>.producer.`

Default values can be set by using the prefix `spring.cloud.stream.default.producer.`

* autoStartup
  * Signals if this consumer needs to be started automatically
  * Default: `true`.
* partitionKeyExpression
  * A SpEL expression that determines how to partition outbound data. If set, outbound data on this binding is partitioned. `partitionCount` must be set to a value greater than 1 to be effective.&#x20;
  * Default: null.
* partitionKeyExtractorName
  * The name of the bean that implements `PartitionKeyExtractorStrategy`. Used to extract a key used to compute the partition id (see 'partitionSelector\*'). Mutually exclusive with 'partitionKeyExpression'.
  * Default: null.
* partitionSelectorName
  * The name of the bean that implements `PartitionSelectorStrategy`. Used to determine partition id based on partition key (see 'partitionKeyExtractor\*'). Mutually exclusive with 'partitionSelectorExpression'.
  * Default: null.
* partitionSelectorExpression
  * A SpEL expression for customizing partition selection. If neither is set, the partition is selected as the `hashCode(key) % partitionCount`, where `key` is computed through either `partitionKeyExpression`.
  * Default: `null`.
* partitionCount
  * The number of target partitions for the data, if partitioning is enabled. Must be set to a value greater than 1 if the producer is partitioned. On Kafka, it is interpreted as a hint. The larger of this and the partition count of the target topic is used instead.
  * Default: `1`.
* requiredGroups
  * A comma-separated list of groups to which the producer must ensure message delivery even if they start after it has been created (for example, by pre-creating durable queues in RabbitMQ).
* headerMode
  * When set to `none`, it disables header embedding on output. It is effective only for messaging middleware that does not support message headers natively and requires header embedding. This option is useful when producing data for non-Spring Cloud Stream applications when native headers are not supported. When set to `headers`, it uses the middleware’s native header mechanism. When set to `embeddedHeaders`, it embeds headers into the message payload.
  * Default: Depends on the binder implementation.
* useNativeEncoding
  * When set to `true`, the outbound message is serialized directly by the client library, which must be configured correspondingly (for example, setting an appropriate Kafka producer value serializer). When this configuration is being used, the outbound message marshalling is not based on the `contentType` of the binding. When native encoding is used, it is the responsibility of the consumer to use an appropriate decoder (for example, the Kafka consumer value de-serializer) to deserialize the inbound message. Also, when native encoding and decoding is used, the `headerMode=embeddedHeaders` property is ignored and headers are not embedded in the message. See the consumer property `useNativeDecoding`.
  * Default: `false`.
* errorChannelEnabled
  * When set to true, if the binder supports asynchroous send results, send failures are sent to an error channel for the destination. See Error Handling for more information.
  * Default: false.

## Binder Properties

这些属性通过 **org.springframework.cloud.stream.config.BinderProperties** 暴露出来，它们必须以 **spring.cloud.stream.binders.\<configurationName>** 为前缀。

* **type**：Binder 类型。通常引用类路径上找到的绑定器之一（META-INF/spring.binders 文件中的键）。默认情况下，它的值与配置名称相同。
* **inheritEnvironment**：配置是否继承应用程序本身的环境。默认值：true。
* **environment**：用于自定义绑定器环境的一组属性的根。当设置了此属性时，创建绑定器的上下文不是应用程序上下文的子上下文。此设置允许在绑定器组件和应用程序组件之间进行完全分离。默认值：空。
* **defaultCandidate**：Binder 配置是否可以被视为默认 binder 或仅在显式引用时使用。此设置允许添加绑定器配置而不干扰默认处理。默认值：true。
