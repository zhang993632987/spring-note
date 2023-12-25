# 在组织服务中编写消息生产者

每当组织数据被添加、更新或删除时，organization 服务将向Kafka主题发布一条消息，表示发生了组织变更事件。发布的消息将包括与变更事件相关联的组织ID以及发生的操作（添加、更新或删除）。

## 添加依赖

在 pom.xml 中，我们需要添加两个依赖项：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-kafka</artifactId>
</dependency>
```

## 绑定消息代理

接下来，需要告诉应用程序它将绑定到一个Spring Cloud Stream消息代理。

{% code overflow="wrap" %}
```java
@RefreshScope
@SpringBootApplication
@MapperScan("com.study.organization.mapper")
public class OrganizationServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrganizationServiceApplication.class, args);
    }

    @Bean
    public Function<String, String> send() {
        return value -> value.toUpperCase();
    }
}
```
{% endcode %}

## 发布消息

下一步是创建发布消息的逻辑：

```java
@Slf4j
@Component
public class SimpleSourceBean {

    private MessageChannel sendOrgChannel;

    public void publishOrganizationChange(ActionEnum action,
                                          String organizationId) {
        log.debug("Sending Kafka message {} for Organization Id: {}",
                action, organizationId);
        // Publishes a Java POJO message
        OrganizationChangeModel change = new OrganizationChangeModel(
                OrganizationChangeModel.class.getTypeName(),
                action.toString(),
                organizationId,
                UserContextHolder.getContext().getCorrelationId()
        );
        // Sends the message from a channel defined in the Source class
        sendOrgChannel.send(MessageBuilder.withPayload(change).build());
    }

    @Lazy
    @Qualifier("send-org")
    @Autowired(required = false)
    public void setSendOrgChannel(MessageChannel sendOrgChannel) {
        this.sendOrgChannel = sendOrgChannel;
    }
}
```

```java
public enum ActionEnum {
   GET,
   CREATED,
   UPDATED,
   DELETED
}
```

The actual publication of the message occurs in the publishOrganizationChange() method. This method builds a Java POJO called OrganizationChangeModel.

> We should always include a correlation ID in our events as it helps greatly with tracking and debugging the flow of messages through our services.

## 配置 binding

Listing 10.6 shows the configuration that maps our service’s Spring Cloud Stream Source to a Kafka message broker and a message topic.

```yaml
spring:
  cloud:
    stream:
      function:
        definition: send
        bindings:
          send-out-0: send-org
      bindings:
        send-org:
          destination: orgChangeTopic
          producer:
            useNativeEncoding: true
      kafka:
        bindings:
          send-org:
            producer:
              configuration:
                value.serializer: org.springframework.kafka.support.serializer.JsonSerializer
        binder:
          brokers: 192.168.10.110:9094
          requiredAcks: all
```

The spring.cloud.stream.bindings is the start of the configuration needed for our service to publish to a Spring Cloud Stream message broker. The configuration property spring.cloud.stream.bindings.output in the listing maps the source.output() channel in listing 10.4 to the orgChangeTopic on the message broker we’re going to communicate with. It also tells Spring Cloud Stream that messages sent to this topic should be serialized as JSON. Spring Cloud Stream can serialize messages in multiple formats including JSON, XML, and the Apache Foundation’s Avro format.

> ## What data should I put in the message?
>
> As you may have noticed, in all our examples, we only return the organization ID of the organization record that’s changed. We never put a copy of the data changes in the message. Also, we use messages based on system events to tell other services that the data state has changed, and we always force the other services to go back to the master (the service that owns the data) to retrieve a new copy of the data. This approach is costlier in terms of execution time, but it guarantees we always have the latest copy of the data. But a slight chance exists that the data we work with could change right after we’ve read it from the source system. That’s much less likely to occur, however, than if we blindly consume the information right off the queue.
>
> To think carefully about how much data you’re passing around. Sooner or later, you’ll run into a situation where the data passed is “stale.” It could be because a problem caused it to sit in the message queue too long, or a previous message containing data failed and the data you’re passing in the message now represents data in an inconsistent state. This might be because your application relied on the message’s state rather than on the actual state in the underlying data store. If you’re going to pass state in your message, make sure to include a date-time stamp or version number so that the service that’s consuming the data can inspect the data passed to it and ensure that it’s not older than the copy of the data it already has.

