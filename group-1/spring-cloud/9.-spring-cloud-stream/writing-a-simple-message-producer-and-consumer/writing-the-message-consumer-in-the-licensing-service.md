# Writing the message consumer in the licensing service

For this example, the licensing service will consume the message published by the organization service.&#x20;

## 添加依赖

To begin, we need to add our Spring Cloud Stream dependencies to the licensing services pom.xml file.

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

Next, we’ll tell the licensing service that it needs to use Spring Cloud Stream to bind to a message broker. Like the organization service, we’ll annotate the licensing service’s bootstrap class, LicenseServiceApplication, with the @EnableBinding annotation. The difference between the licensing service and the organization service is the value we’ll pass to @EnableBinding, which the following listing shows.

```java
@EnableBinding(Sink.class)   
public class LicenseServiceApplication {

   // Executes this method each time a message is received from the input channel
   @StreamListener(Sink.INPUT)    
   public void loggerSink(OrganizationChangeModel orgChange) {
      logger.debug("Received an {} event for organization id {}",
       orgChange.getAction(), orgChange.getOrganizationId());
   }
   //Rest of the code omitted for conciseness
}
```

Because the licensing service is the message consumer, we’ll pass @EnableBinding the value Sink.class. This tells Spring Cloud Stream to bind to a message broker using the default Spring Sink interface. Similar to the Source interface (described in section 10.3.1), Spring Cloud Stream exposes a default channel on the Sink interface. This Sink interface channel is called input and is used to listen for incoming messages.

Once we’ve defined that we want to listen for messages via the @EnableBinding annotation, we can write the code to process a message coming from the Sink input channel. To do this, we’ll use the Spring Cloud Stream @StreamListener annotation. This annotation tells Spring Cloud Stream to execute the loggerSink() method when receiving a message from the input channel. Spring Cloud Stream automatically deserializes the incoming message to a Java POJO called OrganizationChangeModel.

## 配置 channel 到消息代理的映射

the licensing service’s configuration implements the mapping of the message broker’s topic to the input channel.

<pre class="language-properties"><code class="lang-properties"># Maps the input channel to the orgChangeTopic queue
<strong>spring.cloud.stream.bindings.input.destination=orgChangeTopic
</strong>spring.cloud.stream.bindings.input.content-type=application/json
spring.cloud.stream.bindings.input.group=licensingGroup   
spring.cloud.stream.kafka.binder.zkNodes=localhost
spring.cloud.stream.kafka.binder.brokers=localhost
</code></pre>

The configuration in this listing looks like the configuration for the organization service. It has, however, two key differences. First, we now have an input channel defined with the spring.cloud.stream.bindings property. This value maps to the Sink.INPUT channel defined in the code from listing 10.8. This property maps the input channel to the orgChangeTopic. Second, we see the introduction of a new property called spring.cloud.stream.bindings.input.group. The group property defines the name of the consumer group that will consume the message.

The concept of a consumer group is this: we can have multiple services with each service having multiple instances listening to the same message queue. We want each unique service to process a copy of a message, but we only want one service instance within a group of service instances to consume and process a message. The group property identifies the consumer group that the service belongs to.

As long as all the service instances have the same group name, Spring Cloud Stream and the underlying message broker will guarantee that only one copy of the message will be consumed by a service instance belonging to that group. In the case of our licensing service, we’ll call the group property value licensingGroup.

