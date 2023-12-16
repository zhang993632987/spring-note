# Writing the message producer in the organization service

Every time organization data is added, updated, or deleted, the organization service will publish a message to a Kafka topic, indicating that an organization change event has occurred. The published message will include the organizationID associated with the change event and what action occurred (add, update, or delete).

## 添加依赖

In the pom.xml, we need to add two dependencies, one for the core Spring Cloud Stream libraries and the other for the Spring Cloud Stream Kafka libraries:

```
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

Once we’ve defined the Maven dependencies, we need to tell our application that it’s going to bind to a Spring Cloud Stream message broker. We can do this by annotating the organization service’s bootstrap class, OrganizationServiceApplication, with @EnableBinding.

{% code overflow="wrap" %}
```java
@SpringBootApplication
@RefreshScope
@EnableResourceServer
@EnableBinding(Source.class)    // Tells Spring Cloud Stream to bind the application to a message broker
public class OrganizationServiceApplication {
   public static void main(String[] args) {
      SpringApplication.run(OrganizationServiceApplication.class, args);
   }
}
```
{% endcode %}

The @EnableBinding annotation tells Spring Cloud Stream that we want to bind the service to a message broker. The use of Source.class in @EnableBinding tells Spring Cloud Stream that this service will communicate with the message broker via a set of channels defined in the Source class.

Channels sit above a message queue. Spring Cloud Stream has a default set of channels that can be configured to speak to a message broker.

## 发布消息

The next step is to create the logic to publish the message. The following listingshows the code for this class.

```java
@Component
public class SimpleSourceBean {
    private Source source;
    private static final Logger logger =
       LoggerFactory.getLogger(SimpleSourceBean.class);
       
    // Injects a Source interface implementation for use by the service
    public SimpleSourceBean(Source source){   
        this.source = source;
    }
    public void publishOrganizationChange(ActionEnum action, 
                                     String organizationId){
       logger.debug("Sending Kafka message {} for Organization Id: {}",
                    action, organizationId);
        // Publishes a Java POJO message
        OrganizationChangeModel change =  new OrganizationChangeModel(
                OrganizationChangeModel.class.getTypeName(),
                action.toString(),
                organizationId,
                UserContext.getCorrelationId()); 
        // Sends the message from a channel defined in the Source class 
        source.output().send(MessageBuilder  
                          .withPayload(change)
                          .build()); 
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

```properties
# Names the message queue (or topic) that writes the messages
spring.cloud.stream.bindings.output.destination=orgChangeTopic  
# Provides (hints) the message type that’s sent and received (in this case, JSON) 
spring.cloud.stream.bindings.output.content-type=application/json    
spring.cloud.stream.kafka.binder.zkNodes=localhost    
spring.cloud.stream.kafka.binder.brokers=localhost
```

The spring.cloud.stream.bindings is the start of the configuration needed for our service to publish to a Spring Cloud Stream message broker. The configuration property spring.cloud.stream.bindings.output in the listing maps the source.output() channel in listing 10.4 to the orgChangeTopic on the message broker we’re going to communicate with. It also tells Spring Cloud Stream that messages sent to this topic should be serialized as JSON. Spring Cloud Stream can serialize messages in multiple formats including JSON, XML, and the Apache Foundation’s Avro format.

> ## What data should I put in the message?
>
> As you may have noticed, in all our examples, we only return the organization ID of the organization record that’s changed. We never put a copy of the data changes in the message. Also, we use messages based on system events to tell other services that the data state has changed, and we always force the other services to go back to the master (the service that owns the data) to retrieve a new copy of the data. This approach is costlier in terms of execution time, but it guarantees we always have the latest copy of the data. But a slight chance exists that the data we work with could change right after we’ve read it from the source system. That’s much less likely to occur, however, than if we blindly consume the information right off the queue.
>
> To think carefully about how much data you’re passing around. Sooner or later, you’ll run into a situation where the data passed is “stale.” It could be because a problem caused it to sit in the message queue too long, or a previous message containing data failed and the data you’re passing in the message now represents data in an inconsistent state. This might be because your application relied on the message’s state rather than on the actual state in the underlying data store. If you’re going to pass state in your message, make sure to include a date-time stamp or version number so that the service that’s consuming the data can inspect the data passed to it and ensure that it’s not older than the copy of the data it already has.

