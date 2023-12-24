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

In the preceding example, we define a bean of type java.util.function.Function called send to be acting as message handler whose 'input' and 'output' must be bound to the external destinations exposed by the provided destination binder. By default the 'input' and 'output' binding names will be send-in-0 and send-out-0.

So if  you would want to map the input of this function to a remote destination (e.g., topic, queue etc) called "my-topic" you would do so with the following property:

```
spring.cloud.stream.bindings.send-in-0.destination=my-topic
```

> The naming convention used to name input and output bindings is as follows:&#x20;
>
> * input - + -in- + \<index>
> * output - + -out- + \<index>
>
> The in and out corresponds to the type of binding (such as input or output). The index is the index of the input or output binding. It is always 0 for typical single input/output function.

> Descriptive Binding Names
>
> you can map an implicit binding name to an explicit binding name. you can do it with spring.cloud.stream.function.bindings.\<bind\_name> property.
>
> ```
> spring.cloud.stream.function.bindings.send-in-0=input
> ```
>
> In the preceding example you mapped and effectively renamed uppercase-in-0 binding name to input. Now all configuration properties can refer to input binding name instead (e.g., spring.cloud.stream.bindings.input.destination=my-topic).

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

In listing 10.4, we injected the Spring Cloud Source class into our code. Remember, all communication to a specific message topic occurs through a Spring Cloud Stream construct called a channel, which is represented by a Java interface class. In the listing, we used the Source interface, which exposes a single method called output().

The Source interface is convenient to use when our service only needs to publish to a single channel. The output() method returns a class of type MessageChannel. With this type, we’ll send messages to the message broker.

TheActionEnum passed by the parameters in the output() method contains the following actions:&#x20;

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

If we go back to the SimpleSourceBean class, we can see that when we’re ready to publish the message, we can use the send() method on the MessageChannel class returned from the source.output() method like this:

```java
source.output().send(MessageBuilder.withPayload(change).build());
```

The send() method takes a Spring Message class. We use a Spring helper class, called MessageBuilder, to take the contents of the OrganizationChangeModel class and convert it to a Spring Message class.

## 配置 channel

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

