# Writing the message consumer in the licensing service

For this example, the licensing service will consume the message published by the organization service.&#x20;

## 添加依赖

To begin, we need to add our Spring Cloud Stream dependencies to the licensing services pom.xml file.

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

```java
@Bean
public Consumer<OrganizationChangeModel> consumer() {
    return orgChange -> log.debug("Received an {} event for organization id {}",
                orgChange.getAction(), orgChange.getOrganizationId());
}
```

## 配置 binding

```yaml
spring:
  cloud:
    stream:
      function:
        definition: consumer
        bindings:
          consumer-in-0: consumer-org
      bindings:
        consumer-org:
          destination: orgChangeTopic
          consumer:
            useNativeEncoding: true
      kafka:
        bindings:
          consumer-org:
            consumer:
              configuration:
                value.deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
          brokers: 192.168.10.110:9094
          requiredAcks: all
```
