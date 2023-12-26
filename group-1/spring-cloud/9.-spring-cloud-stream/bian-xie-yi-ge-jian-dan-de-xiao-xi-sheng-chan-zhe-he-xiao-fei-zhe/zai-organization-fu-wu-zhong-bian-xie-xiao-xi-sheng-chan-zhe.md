# 在organization服务中编写消息生产者

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

    private StreamBridge streamBridge;

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
        streamBridge.send("send-out-0", change);
    }

    @Autowired
    public void setStreamBridge(StreamBridge streamBridge) {
        this.streamBridge = streamBridge;
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

消息的实际发布发生在 publishOrganizationChange() 方法中。该方法构建一个名为 OrganizationChangeModel 的 Java POJO。

> ### <mark style="color:orange;">**在事件中始终包含关联 ID 对于通过我们的服务跟踪和调试消息流非常有帮助。**</mark>

## 配置 binding

```yaml
spring:
  cloud:
    stream:
      function:
        definition: send
      bindings:
        send-out-0:
          destination: orgChangeTopic
          contentType: application/json
          group: organization
      kafka:
        binder:
          brokers: 192.168.10.110:9094
          requiredAcks: all
```

> ## 在消息中放入什么数据？
>
> **在上面的示例中，仅返回了发生更改的组织记录的组织 ID，**从未在消息中放入数据更改的副本。此外，基于系统事件告知其他服务数据状态已更改，并**始终强制其他服务返回主服务（拥有数据的服务）检索数据的新副本**。
>
> * 从执行时间的角度来看，这种方法更昂贵，但它保证我们始终拥有数据的最新副本。但存在这样一种可能性，在从源系统读取数据后，数据可能会在处理期间发生变化。但是，这种情况发生的可能性较小，远不及直接从队列中获取信息的可能性大。
> * 需要仔细考虑传递的数据量和迟早会遇到的数据“陈旧”问题。这可能是因为问题导致它在消息队列中停留的时间太长，或者之前包含数据的消息发送失败，现在传递的数据表示不一致状态的数据。如果将数据传递给消息，请确保包含日期时间戳或版本号，以便消费数据的服务可以检查传递给它的数据，并确保其不比其已有的数据副本更旧。
>
> **在消息中放入什么数据取决于系统的具体要求和约束。**
