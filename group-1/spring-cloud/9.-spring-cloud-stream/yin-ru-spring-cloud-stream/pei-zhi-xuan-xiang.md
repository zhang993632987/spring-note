# 配置选项

## Binding Service Properties <a href="#_binding_service_properties" id="_binding_service_properties"></a>

These properties are exposed via `org.springframework.cloud.stream.config.BindingServiceProperties`

* spring.cloud.stream.instanceCount
  * The number of deployed instances of an application. Must be set for partitioning on the producer side. Must be set on the consumer side when using RabbitMQ and with Kafka if `autoRebalanceEnabled=false`.
  * Default: `1`.
* spring.cloud.stream.instanceIndex
  * The instance index of the application: A number from `0` to `instanceCount - 1`. Used for partitioning with RabbitMQ and with Kafka if `autoRebalanceEnabled=false`.&#x20;
  * Automatically set in Cloud Foundry to match the application’s instance index.
* spring.cloud.stream.dynamicDestinations
  * A list of destinations that can be bound dynamically (for example, in a dynamic routing scenario). If set, only listed destinations can be bound.
  * Default: empty (letting any destination be bound).
* spring.cloud.stream.defaultBinder
  * The default binder to use, if multiple binders are configured.
  * Default: empty.
* spring.cloud.stream.overrideCloudConnectors
  * This property is only applicable when the `cloud` profile is active and Spring Cloud Connectors are provided with the application. If the property is `false` (the default), the binder detects a suitable bound service (for example, a RabbitMQ service bound in Cloud Foundry for the RabbitMQ binder) and uses it for creating connections (usually through Spring Cloud Connectors). When set to `true`, this property instructs binders to completely ignore the bound services and rely on Spring Boot properties (for example, relying on the `spring.rabbitmq.*` properties provided in the environment for the RabbitMQ binder). The typical usage of this property is to be nested in a customized environment [when connecting to multiple systems](https://docs.spring.io/spring-cloud-stream/docs/4.0.4-SNAPSHOT/reference/html/spring-cloud-stream.html#multiple-systems).
  * Default: `false`.
* spring.cloud.stream.bindingRetryInterval
  * The interval (in seconds) between retrying binding creation when, for example, the binder does not support late binding and the broker (for example, Apache Kafka) is down. Set it to zero to treat such conditions as fatal, preventing the application from starting.
  * Default: `30`

## Binding Properties <a href="#binding-properties" id="binding-properties"></a>

Binding properties are supplied by using the format of `spring.cloud.stream.bindings.<bindingName>.<property>=<value>`. The `<bindingName>` represents the name of the binding being configured.

To avoid repetition, Spring Cloud Stream supports setting values for all bindings, in the format of `spring.cloud.stream.default.<property>=<value>` and `spring.cloud.stream.default.<producer|consumer>.<property>=<value>` for common binding properties.

### **Common Binding Properties**

These properties are exposed via `org.springframework.cloud.stream.config.BindingProperties`

The following binding properties are available for both input and output bindings and must be prefixed with `spring.cloud.stream.bindings.<bindingName>.`

Default values can be set by using the `spring.cloud.stream.default` prefix(for example `spring.cloud.stream.default.contentType=application/json`).

* destination
  * The target destination of a binding on the bound middleware (for example, the RabbitMQ exchange or Kafka topic). If binding represents a consumer binding (input), it could be bound to multiple destinations, and the destination names can be specified as comma-separated `String` values. If not, the actual binding name is used instead.&#x20;
  * The default value of this property cannot be overridden.
* group
  * The consumer group of the binding. Applies only to inbound bindings.&#x20;
  * Default: `null` (indicating an anonymous consumer).
* contentType
  * The content type of this binding.&#x20;
  * Default: `application/json`.
* binder
  * The binder used by this binding.&#x20;
  * Default: `null` (the default binder is used, if it exists).

### **Consumer Properties**

These properties are exposed via `org.springframework.cloud.stream.binder.ConsumerProperties.`

The following binding properties are available for input bindings only and must be prefixed with `spring.cloud.stream.bindings.<bindingName>.consumer.`

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

### **Producer Properties**

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
