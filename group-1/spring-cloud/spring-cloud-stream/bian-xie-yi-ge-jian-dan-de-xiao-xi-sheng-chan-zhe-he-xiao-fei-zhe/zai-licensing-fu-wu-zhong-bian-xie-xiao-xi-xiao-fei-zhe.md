# 在licensing服务中编写消息消费者

## 添加依赖

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
public Consumer<OrganizationChangeModel> consume() {
    return orgChange -> log.debug("Received an {} event for organization id {}",
                orgChange.getAction(), orgChange.getOrganizationId());
}
```

## 配置 binding

```yaml
spring:
  cloud:
    function:
      definition: consume
    stream:
      bindings:
        consume-in-0:
          destination: orgChangeTopic
          contentType: application/json
          group: license
      kafka:
        brokers: 192.168.10.110:9094
        requiredAcks: all
```
