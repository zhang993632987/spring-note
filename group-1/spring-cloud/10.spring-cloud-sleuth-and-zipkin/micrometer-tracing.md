# Micrometer Tracing

> The Spring team realized that tracing could be separated from Spring Cloud and created the Micrometer Tracing project, which is, essentially, a Spring-agnostic copy of Spring Cloud Sleuth.
>
> Spring Boot Actuator provides dependency management and auto-configuration for Micrometer Tracing, a facade for popular tracer libraries.

## Adding Spring Cloud Sleuth to licensing and organization

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>
```

You can include the current trace and span id in the logs by setting the **logging.pattern.level** property to **%5p \[${spring.application.name:},%X{traceId:-},%X{spanId:-}]**

```yaml
logging:
  pattern:
    level: '%5p [${spring.application.name:},%X{traceId:-},%X{spanId:-}]'
```

These dependencies pull in all the core libraries needed for Spring Cloud Sleuth. Once they are pulled in, our service will now

* Inspect every incoming HTTP service and determine whether Spring Cloud Sleuth tracing information exists in the incoming call. If the Spring Cloud Sleuth tracing data is present, the tracing information passed into our microservice will be captured and made available to our service for logging and processing.&#x20;
* Add Spring Cloud Sleuth tracing information to the Spring MDC so that every log statement created by our microservice will be added to the log.&#x20;

{% hint style="info" %}
## <mark style="color:blue;">提示</mark>

To automatically propagate traces over the network, use the auto-configured **RestTemplateBuilder** or **WebClient.Builder** to construct the client.&#x20;

If you create the `WebClient` or the `RestTemplate` without using the auto-configured builders, automatic trace propagation won’t work!
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

```xml
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-micrometer</artifactId>
</dependency>
```
{% endhint %}

## Anatomy of a Spring Cloud Sleuth trace

If everything is set up correctly, any log statements written in our service application code will now include Spring Cloud Sleuth trace information. For example, figure 11.1 shows what the service’s output would look like if we were to issue an HTTP GET on the following endpoint in the organization service:

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Spring Cloud Sleuth adds four pieces of information to each log entry. These four pieces are as follows:

1. The application name of the service where the log entry is entered.&#x20;
2. The trace ID, which is the equivalent term for correlation ID. This is a unique number that represents an entire transaction.
3. The span ID, which is a unique ID that represents part of the overall transaction. Each service participating within the transaction will have its own span ID. Span IDs are particularly relevant if you integrate with Zipkin to visualize your transactions.
