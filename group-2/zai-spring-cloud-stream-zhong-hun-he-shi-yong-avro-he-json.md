# 在Spring Cloud Stream中混合使用avro和JSON

**在 Spring Cloud Stream 中混合使用 avro 序列化 和 普通的 JSON 序列化时遇到了一个奇怪的问题。**

## 主要代码

<details>

<summary><mark style="color:purple;"><strong>yaml配置</strong></mark></summary>

```yaml
  spring:
    cloud:
      function:
        definition: send;sendAvro
      stream:
        bindings:
          send-in-0:
            autoStartup: false
            group: organization
          sendAvro-in-0:
            autoStartup: false
            group: organization
          send-out-0:
            destination: orgChangeTopic
            contentType: application/json
            producer:
              useNativeEncoding: false
          sendAvro-out-0:
            destination: orgChangeTopicAvro
            producer:
              useNativeEncoding: true
        kafka:
          bindings:
            sendAvro-out-0:
              producer:
                configuration:
                  schema.registry.url: http://kubernetes:8074
                  value.serializer: io.confluent.kafka.serializers.KafkaAvroSerializer
          binder:
            brokers: kubernetes:9094
            requiredAcks: all
```

</details>

<details>

<summary><mark style="color:purple;"><strong>发送消息的代码</strong></mark></summary>

```java
@Slf4j
@Component
public class SimpleSourceBean {

    private StreamBridge streamBridge;

    public void publishOrganizationChange(ActionEnum action,
                                          String organizationId,
                                          boolean avro) {
        log.debug("Sending Kafka message {} for Organization Id: {}",
                action, organizationId);
        if (avro) {
            OrganizationChangeModel change = new OrganizationChangeModel(
                    OrganizationChangeModel.class.getTypeName(),
                    action.toString(),
                    organizationId,
                    UserContextHolder.getContext().getCorrelationId()
            );
            // Sends the message from a channel defined in the Source class
            streamBridge.send("sendAvro-out-0", change);
        } else {
            // Publishes a Java POJO message
            OrganizationChangeModel2 change = new OrganizationChangeModel2(
                    OrganizationChangeModel2.class.getTypeName(),
                    action.toString(),
                    organizationId,
                    UserContextHolder.getContext().getCorrelationId()
            );
            // Sends the message from a channel defined in the Source class
            streamBridge.send("send-out-0", change);
        }
    }

    @Autowired
    public void setStreamBridge(StreamBridge streamBridge) {
        this.streamBridge = streamBridge;
    }
}
```

</details>

在上述代码中，根据 avro 的值，发送消息到不同的主题，两个主题中的消息使用了不同的序列化方式：

* **send-out-0 使用普通的 JSON 序列化，由 Spring 的 MessageConverter 机制进行处理**
* **而 sendAvro-out-0 使用 kafka 生产者的 value serializer 进行序列化**

## 问题

发送消息的次序不同，产生了不同的异常：

<details>

<summary><strong>先使用 send-out-0 发送消息，后使用 sendAvro-out-0 发送消息</strong>，遇到的异常</summary>

{% code overflow="wrap" lineNumbers="true" %}
```log
java.lang.NullPointerException: Cannot invoke "Object.getClass()" because "result" is null 
at org.springframework.cloud.function.cloudevent.CloudEventsFunctionInvocationHelper.doPostProcessResult(CloudEventsFunctionInvocationHelper.java:138) ~[spring-cloud-function-context-4.0.5.jar:4.0.5]
at org.springframework.cloud.function.cloudevent.CloudEventsFunctionInvocationHelper.postProcessResult(CloudEventsFunctionInvocationHelper.java:114) ~[spring-cloud-function-context-4.0.5.jar:4.0.5]
at org.springframework.cloud.function.cloudevent.CloudEventsFunctionInvocationHelper.postProcessResult(CloudEventsFunctionInvocationHelper.java:48) ~[spring-cloud-function-context-4.0.5.jar:4.0.5]
at org.springframework.cloud.stream.function.StreamBridge.send(StreamBridge.java:185) ~[spring-cloud-stream-4.0.4.jar:4.0.4]
at org.springframework.cloud.stream.function.StreamBridge.send(StreamBridge.java:146) ~[spring-cloud-stream-4.0.4.jar:4.0.4]
at org.springframework.cloud.stream.function.StreamBridge.send(StreamBridge.java:141) ~[spring-cloud-stream-4.0.4.jar:4.0.4]
at com.study.organization.message.SimpleSourceBean.publishOrganizationChange(SimpleSourceBean.java:35) ~[classes/:na]
...
```
{% endcode %}

</details>

<details>

<summary>先使用 sendAvro-out-0 发送消息，后使用 send-out-0 发送消息，遇到的异常</summary>

{% code overflow="wrap" lineNumbers="true" %}
```log
java.lang.ClassCastException: class com.study.organization.model.OrganizationChangeModel2 cannot be cast to class [B (com.study.organization.model.OrganizationChangeModel2 is in unnamed module of loader org.springframework.boot.devtools.restart.classloader.RestartClassLoader @703cc83; [B is in module java.base of loader 'bootstrap')
	at org.apache.kafka.common.serialization.ByteArraySerializer.serialize(ByteArraySerializer.java:19) ~[kafka-clients-3.4.1.jar:na]
	at org.apache.kafka.common.serialization.Serializer.serialize(Serializer.java:62) ~[kafka-clients-3.4.1.jar:na]
	at org.apache.kafka.clients.producer.KafkaProducer.doSend(KafkaProducer.java:1015) ~[kafka-clients-3.4.1.jar:na]
	at org.apache.kafka.clients.producer.KafkaProducer.send(KafkaProducer.java:962) ~[kafka-clients-3.4.1.jar:na]
	at org.springframework.kafka.core.DefaultKafkaProducerFactory$CloseSafeProducer.send(DefaultKafkaProducerFactory.java:1062) ~[spring-kafka-3.0.13.jar:3.0.13]
	at org.springframework.kafka.core.KafkaTemplate.doSend(KafkaTemplate.java:785) ~[spring-kafka-3.0.13.jar:3.0.13]
	at org.springframework.kafka.core.KafkaTemplate.observeSend(KafkaTemplate.java:754) ~[spring-kafka-3.0.13.jar:3.0.13]
	at org.springframework.kafka.core.KafkaTemplate.send(KafkaTemplate.java:564) ~[spring-kafka-3.0.13.jar:3.0.13]
	at org.springframework.integration.kafka.outbound.KafkaProducerMessageHandler.handleRequestMessage(KafkaProducerMessageHandler.java:532) ~[spring-integration-kafka-6.1.5.jar:6.1.5]
	at org.springframework.integration.handler.AbstractReplyProducingMessageHandler.handleMessageInternal(AbstractReplyProducingMessageHandler.java:136) ~[spring-integration-core-6.1.5.jar:6.1.5]
	at org.springframework.integration.handler.AbstractMessageHandler.doHandleMessage(AbstractMessageHandler.java:105) ~[spring-integration-core-6.1.5.jar:6.1.5]
	at org.springframework.integration.handler.AbstractMessageHandler.handleMessage(AbstractMessageHandler.java:73) ~[spring-integration-core-6.1.5.jar:6.1.5]
	at org.springframework.cloud.stream.binder.kafka.KafkaMessageChannelBinder$ProducerConfigurationMessageHandler.handleMessage(KafkaMessageChannelBinder.java:1594) ~[spring-cloud-stream-binder-kafka-4.0.4.jar:4.0.4]
	at org.springframework.cloud.stream.binder.AbstractMessageChannelBinder$SendingHandler.handleMessageInternal(AbstractMessageChannelBinder.java:1185) ~[spring-cloud-stream-4.0.4.jar:4.0.4]
	at org.springframework.integration.handler.AbstractMessageHandler.doHandleMessage(AbstractMessageHandler.java:105) ~[spring-integration-core-6.1.5.jar:6.1.5]
	at org.springframework.integration.handler.AbstractMessageHandler.handleMessage(AbstractMessageHandler.java:73) ~[spring-integration-core-6.1.5.jar:6.1.5]
	at org.springframework.integration.dispatcher.AbstractDispatcher.tryOptimizedDispatch(AbstractDispatcher.java:115) ~[spring-integration-core-6.1.5.jar:6.1.5]
	at org.springframework.integration.dispatcher.UnicastingDispatcher.doDispatch(UnicastingDispatcher.java:133) ~[spring-integration-core-6.1.5.jar:6.1.5]
	at org.springframework.integration.dispatcher.UnicastingDispatcher.dispatch(UnicastingDispatcher.java:106) ~[spring-integration-core-6.1.5.jar:6.1.5]
	at org.springframework.integration.channel.AbstractSubscribableChannel.doSend(AbstractSubscribableChannel.java:72) ~[spring-integration-core-6.1.5.jar:6.1.5]
	at org.springframework.integration.channel.AbstractMessageChannel.sendInternal(AbstractMessageChannel.java:375) ~[spring-integration-core-6.1.5.jar:6.1.5]
	at org.springframework.integration.channel.AbstractMessageChannel.sendWithMetrics(AbstractMessageChannel.java:346) ~[spring-integration-core-6.1.5.jar:6.1.5]
	at org.springframework.integration.channel.AbstractMessageChannel.send(AbstractMessageChannel.java:326) ~[spring-integration-core-6.1.5.jar:6.1.5]
	at org.springframework.integration.channel.AbstractMessageChannel.send(AbstractMessageChannel.java:299) ~[spring-integration-core-6.1.5.jar:6.1.5]
	at org.springframework.cloud.stream.function.StreamBridge.send(StreamBridge.java:187) ~[spring-cloud-stream-4.0.4.jar:4.0.4]
	at org.springframework.cloud.stream.function.StreamBridge.send(StreamBridge.java:146) ~[spring-cloud-stream-4.0.4.jar:4.0.4]
	at org.springframework.cloud.stream.function.StreamBridge.send(StreamBridge.java:141) ~[spring-cloud-stream-4.0.4.jar:4.0.4]
	at com.study.organization.message.SimpleSourceBean.publishOrganizationChange(SimpleSourceBean.java:45) ~[classes/:na]
	...
```
{% endcode %}

</details>

## 分析

为了找出上述问题的原因，使用 IDEA 的 DEBUG 对代码进行了追踪，最后发现**出现问题的原因在于 StreamBridge 中的 getStreamBridgeFunction 方法：**

```java
  private synchronized FunctionInvocationWrapper getStreamBridgeFunction(String outputContentType, ProducerProperties producerProperties) {
     if (StringUtils.hasText(outputContentType) && this.streamBridgeFunctionCache.containsKey(outputContentType)) {
        return this.streamBridgeFunctionCache.get(outputContentType);
     }
     else {
        FunctionInvocationWrapper functionToInvoke = this.functionCatalog.lookup(STREAM_BRIDGE_FUNC_NAME, outputContentType.toString());
        this.streamBridgeFunctionCache.put(outputContentType, functionToInvoke);
        functionToInvoke.setSkipOutputConversion(producerProperties.isUseNativeEncoding());
        return functionToInvoke;
     }
  }
```

<mark style="color:orange;">**上述代码中使用了一个名为 streamBridgeFunctionCache 的内部缓存，缓存的键是 outputContentType，而 avro 和 JSON 序列化方式的 outputContentType 值都是 application/json**</mark>，最重要的是 skipOutputConversion 字段，两种序列化方式理应是完全不一样的，但是缓存的命中违背了这一要求！

skipOutputConversion 决定了是否对消息的载荷（payload）使用 MessageConverter 进行处理，avro 不能使用该方式进行处理，而 JSON 需要使用 MessageConverter 将对象转换成二进制数组。

## 消息序列化机制

根据上述内容，可以分析 StreamBridge 中消息序列化的机制如下：

1. 首先，**判断 binding 的 useNativeEncoding 值**，该配置值赋给了 FunctionInvocationWrapper 的 skipOutputConversion
   1. **如果 useNativeEncoding 为 false，则使用 MessageConverter 将消息的载荷序列化成二进制数组**（由 JsonMapper 完成）
   2. 如果 useNativeEncoding 为 true，不对消息载荷进行处理。
2. 然后，**将消息 Message 交给 KafkaTemplate，由 KafkaTemplate 使用配置的 value.serializer 进行序列化**，如果没有设置，**默认为 ByteArraySerializer**。

## 解决方案

根据上述对问题的分析，只需要名为 sendAvro-out-0 的 binding 下增加 contentType 的配置即可解决问题：

```yaml
sendAvro-out-0:
  destination: orgChangeTopicAvro
  contentType: text/plain
  producer:
    useNativeEncoding: true
```

增加 contentType 配置后，便可以不命中缓存。
