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
* **maxAttempts**：处理消息的尝试次数（包括第一次）。设置为 1 以禁用重试。默认值：3。
* **backOffInitialInterval**：初始重试间隔。默认值：1000。
* **backOffMaxInterval**：最大重试间隔。默认值：10000。
* **backOffMultiplier**：重试间隔的倍增因子。 默认值：2.0。
* **defaultRetryable**：监听器抛出的未在 retryableExceptions 中列出的异常是否可重试。默认值：true。
* retryableExceptions：一个以 Throwable 类名为键、布尔值为值的映射。指定那些异常（及其子类）将或不将被重试。示例：spring.cloud.stream.bindings.input.consumer.retryableExceptions.java.lang.IllegalStateException=false。 默认值：空。
* **instanceCount**：当设置为大于等于零的值时，允许自定义此消费者的实例数（如果与 spring.cloud.stream.instanceCount 不同）。当设置为负值时，默认为 spring.cloud.stream.instanceCount。默认值：-1。
* **instanceIndex**：当设置为大于等于零的值时，允许自定义此消费者的实例索引（如果与 spring.cloud.stream.instanceIndex 不同）。当设置为负值时，默认为 spring.cloud.stream.instanceIndex。如果提供了 instanceIndexList，则忽略。默认值：-1。
* **instanceIndexList**：与不支持本机分区的绑定器一起使用（例如 RabbitMQ）；允许应用程序实例从多个分区中消费。默认值：空。
* **useNativeDecoding**：**当设置为 true 时，入站消息将由客户端库直接反序列化**，必须相应地进行配置（例如，设置适当的 Kafka 生产者反序列化器）。当使用此配置时，入站消息的解码不基于绑定的 contentType。
  * 使用 native 编码和解码时，生产者负责使用适当的编码器（例如，Kafka 生产者值序列化器）对出站消息进行序列化。
  * 使用 native 编码和解码时，将忽略 headerMode=embeddedHeaders 属性，标头不嵌入到消息中。参见生产者属性 useNativeEncoding。
  * 默认值：false。
* **multiplex**：当设置为 true 时，底层绑定器将在相同的输入绑定上本地多路复用目的地。默认值：false。

> **可以使用 spring.cloud.stream.default.consumer 前缀设置默认值。**

### 生产者属性

这些属性通过 **org.springframework.cloud.stream.binder.ProducerProperties** 暴露出来。以下是仅适用于输出绑定的绑定属性，必须以 **spring.cloud.stream.bindings.\<bindingName>.producer** 为前缀：

* **autoStartup**：指示是否自动启动此生产者。默认值：true。
* **partitionKeyExpression**：一个 SpEL 表达式，确定如何对出站数据进行分区。
  * 如果设置，此绑定上的出站数据将进行分区。
  * 必须将 partitionCount 设置为大于1的值才能生效。
  * 默认值：null。
* **partitionKeyExtractorName**：实现 PartitionKeyExtractorStrategy 的 bean 的名称。
  * 用于提取用于计算分区ID的键（参见 'partitionSelector\*'）。
  * 与 'partitionKeyExpression' 互斥。
  * 默认值：null。
* **partitionSelectorName**：实现 PartitionSelectorStrategy 的 bean 的名称。
  * 用于根据分区键确定分区ID（参见 'partitionKeyExtractor\*'）。
  * 与 'partitionSelectorExpression' 互斥。
  * 默认值：null。
* **partitionSelectorExpression**：用于自定义分区选择的 SpEL 表达式。
  * 如果都未设置，则分区将选择为 hashCode(key) % partitionCount，其中 key 是通过 partitionKeyExpression 计算的。
  * 默认值：null。
* **partitionCount**：如果启用了分区，数据的目标分区数。如果生产者进行了分区，必须将其设置为大于1的值。
  * 在 Kafka 上，partitionCount 被解释为提示。实际使用的值是 partitionCount 和目标主题的分区计数中的较大者。
  * 默认值：1。
* **requiredGroups**：生产者必须确保消息传递到其中的组的逗号分隔列表，即使它们在创建后才开始（例如，通过在 RabbitMQ 中预先创建持久队列）。
* **headerMode**：当设置为 none 时，在输出上禁用标头嵌入。仅对原生不支持消息标头且需要嵌入标头的消息中间件有效。当为非 Spring Cloud Stream 应用程序生成数据时，原生标头不受支持时，此选项很有用。当设置为 headers 时，它使用中间件的本机标头机制。当设置为 embeddedHeaders 时，它将标头嵌入到消息有效载荷中。默认值：取决于绑定器实现。
* **useNativeEncoding**：**当设置为 true 时，出站消息将由客户端库直接序列化**，必须相应地进行配置（例如，设置适当的 Kafka 生产者值序列化器）。
  * 使用此配置时，出站消息的编组不基于绑定的 contentType。
  * 当使用 native 编码时，消费者有责任使用适当的解码器（例如，Kafka 消费者值反序列化器）对入站消息进行反序列化。
  * 此外，当使用 native 编码和解码时，忽略 headerMode=embeddedHeaders 属性，标头不嵌入到消息中。参见消费者属性 useNativeDecoding。默认值：false。
* **errorChannelEnabled**：当设置为 true 时，如果绑定器支持异步发送结果，则发送失败将被发送到目的地的错误通道。有关更多信息，请参见错误处理。默认值：false。

> **可以使用 spring.cloud.stream.default.producer 前缀设置默认值。**

## Binder Properties

这些属性通过 **org.springframework.cloud.stream.config.BinderProperties** 暴露出来，它们必须以 **spring.cloud.stream.binders.\<configurationName>** 为前缀。

* **type**：Binder 类型。通常引用类路径上找到的绑定器之一（META-INF/spring.binders 文件中的键）。默认情况下，它的值与配置名称相同。
* **inheritEnvironment**：配置是否继承应用程序本身的环境。默认值：true。
* **environment**：用于自定义绑定器环境的一组属性的根。当设置了此属性时，创建绑定器的上下文不是应用程序上下文的子上下文。此设置允许在绑定器组件和应用程序组件之间进行完全分离。默认值：空。
* **defaultCandidate**：Binder 配置是否可以被视为默认 binder 或仅在显式引用时使用。此设置允许添加绑定器配置而不干扰默认处理。默认值：true。
