# Binders

Spring Cloud Stream provides a Binder abstraction for use in connecting to physical destinations at the external middleware.&#x20;

## Producers and Consumers <a href="#_producers_and_consumers" id="_producers_and_consumers"></a>

The following image shows the general relationship of producers and consumers:

<figure><img src="https://docs.spring.io/spring-cloud-stream/docs/4.0.4-SNAPSHOT/reference/html/images/producers-consumers.png" alt=""><figcaption></figcaption></figure>

A producer is any component that sends messages to a binding destination. The binding destination can be bound to an external message broker with a `Binder` implementation for that broker. When invoking the `bindProducer()` method, the first parameter is the name of the destination within the broker, the second parameter is the instance if local destination to which the producer sends messages, and the third parameter contains properties (such as a partition key expression) to be used within the adapter that is created for that binding destination.

A consumer is any component that receives messages from the binding destination. As with a producer, the consumer can be bound to an external message broker. When invoking the `bindConsumer()` method, the first parameter is the destination name, and a second parameter provides the name of a logical group of consumers. Each group that is represented by consumer bindings for a given destination receives a copy of each message that a producer sends to that destination. If there are multiple consumer instances bound with the same group name, then messages are load-balanced across those consumer instances so that each message sent by a producer is consumed by only a single consumer instance within each group.

## Binder Detection <a href="#_binder_detection" id="_binder_detection"></a>

### **Classpath Detection**

By default, Spring Cloud Stream relies on Spring Boot’s auto-configuration to configure the binding process. If a single Binder implementation is found on the classpath, Spring Cloud Stream automatically uses it. For example, a Spring Cloud Stream project that aims to bind only to RabbitMQ can add the following dependency:

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
```

### Multiple Binders on the Classpath <a href="#multiple-binders" id="multiple-binders"></a>

When multiple binders are present on the classpath, the application must indicate which binder is to be used for each destination binding. Binder selection can either be performed globally, using the `spring.cloud.stream.defaultBinder` property (for example, `spring.cloud.stream.defaultBinder=rabbit`) or individually, by configuring the binder on each binding. For instance, a processor application (that has bindings named `input` and `output` for read and write respectively) that reads from Kafka and writes to RabbitMQ can specify the following configuration:

```
spring.cloud.stream.bindings.input.binder=kafka
spring.cloud.stream.bindings.output.binder=rabbit
```

### Connecting to Multiple Systems <a href="#multiple-systems" id="multiple-systems"></a>

By default, binders share the application’s Spring Boot auto-configuration, so that one instance of each binder found on the classpath is created. If your application should connect to more than one broker of the same type, you can specify multiple binder configurations, each with different environment settings.

> Turning on explicit binder configuration disables the default binder configuration process altogether. If you do so, all binders in use must be included in the configuration. Frameworks that intend to use Spring Cloud Stream transparently may create binder configurations that can be referenced by name, but they do not affect the default binder configuration. In order to do so, a binder configuration may have its `defaultCandidate` flag set to false (for example, `spring.cloud.stream.binders.<configurationName>.defaultCandidate=false`). This denotes a configuration that exists independently of the default binder configuration process.

The following example shows a typical configuration for a processor application that connects to two RabbitMQ broker instances:

```yml
spring:
  cloud:
    stream:
      bindings:
        input:
          destination: thing1
          binder: rabbit1
        output:
          destination: thing2
          binder: rabbit2
      binders:
        rabbit1:
          type: rabbit
          environment:
            spring:
              rabbitmq:
                host: <host1>
        rabbit2:
          type: rabbit
          environment:
            spring:
              rabbitmq:
                host: <host2>
```

To activate a specific profile for the particular binder environment, you should use a `spring.profiles.active` property:

```yaml
environment:
    spring:
        profiles:
           active: myBinderProfile
```

## Binding visualization and control <a href="#binding_visualization_control" id="binding_visualization_control"></a>

Spring Cloud Stream supports visualization and control of the Bindings through Actuator endpoints.

You must also enable the `bindings` actuator endpoints by setting the following property: `management.endpoints.web.exposure.include=bindings`.

Once those prerequisites are satisfied. you should see the following in the logs when application start:

```
: Mapped "{[/actuator/bindings/{name}],methods=[POST]. . .
: Mapped "{[/actuator/bindings],methods=[GET]. . .
: Mapped "{[/actuator/bindings/{name}],methods=[GET]. . .
```

To visualize the current bindings, access the following URL: `http://<host>:<port>/actuator/bindings.`

## Binder Configuration Properties <a href="#_binder_configuration_properties" id="_binder_configuration_properties"></a>

The following properties are available when customizing binder configurations. These properties exposed via `org.springframework.cloud.stream.config.BinderProperties.`

They must be prefixed with `spring.cloud.stream.binders.<configurationName>`.

* type
  * The binder type. It typically references one of the binders found on the classpath — in particular, a key in a `META-INF/spring.binders` file.
  * By default, it has the same value as the configuration name.
* inheritEnvironment
  * Whether the configuration inherits the environment of the application itself.
  * Default: `true`.
* environment

Root for a set of properties that can be used to customize the environment of the binder. When this property is set, the context in which the binder is being created is not a child of the application context. This setting allows for complete separation between the binder components and the application components.

Default: `empty`.

defaultCandidate

Whether the binder configuration is a candidate for being considered a default binder or can be used only when explicitly referenced. This setting allows adding binder configurations without interfering with the default processing.

Default: `true`.
