# 分布式追踪

分布式追踪涉及提供一个图形化的展示，展示事务在不同微服务之间是如何流动的。分布式追踪工具还可以提供对单个微服务响应时间的粗略估计。&#x20;

Zipkin 是一个分布式追踪平台，允许我们追踪跨多个服务调用的事务。它让我们以图形方式看到事务所需的时间，并分解了调用涉及的每个微服务所花费的时间。Zipkin对于在微服务架构中识别性能问题非常有价值。

设置Micrometer Tracing和Zipkin包括：

1. 将Micrometer Tracing和Zipkin的JAR文件添加到需要捕获追踪数据的服务中。
2. 在每个服务中配置一个Spring属性，指向将收集追踪数据的Zipkin服务器。
3. 安装和配置Zipkin服务器以收集数据。
4. 定义每个客户端将使用的采样策略，以将追踪信息发送到Zipkin。

## 添加依赖

```xml
<dependency>
    <groupId>io.zipkin.reporter2</groupId>
    <artifactId>zipkin-reporter-brave</artifactId>
</dependency>
```

## 指向收集追踪数据的Zipkin服务器Zipkin服务器

```yaml
management:
  zipkin:
    tracing:
      endpoint: http://kubernetes:9411/api/v2/spans
```

> 可以通过RabbitMQ或Kafka将追踪数据发送到Zipkin服务器。
>
> * 从功能的角度来看，使用HTTP、RabbitMQ或Kafka，Zipkin的行为没有区别。**对于HTTP追踪，Zipkin使用异步线程发送性能数据。**
> * 使用RabbitMQ或Kafka收集追踪数据的主要优势是，如果Zipkin服务器宕机，发送到Zipkin的追踪消息将被“排队”，直到Zipkin可以恢复并接收数据。

## 配置Zipkin服务器

要运行Zipkin服务器，只需要进行少量配置。运行Zipkin时，需要配置的少数几件事之一是Zipkin使用的后端数据存储，Zipkin支持四种不同的后端数据存储：

* 内存数据
* MySQL
* Cassandra
* Elasticsearch

默认情况下，Zipkin使用内存数据存储进行存储。

在这个例子中，将演示如何使用Elasticsearch作为数据存储。为此，需要添加的唯一额外设置是在配置文件的环境部分中添加STORAGE\_TYPE和ES\_HOSTS变量，如下所示：

```yaml
zipkin: 
  restart: always
  image: openzipkin/zipkin:2.25
  environment:
    - STORAGE_TYPE=elasticsearch
    - ES_HOSTS=elasticsearch:9200
  ports:
    - 9411:9411
  depends_on:
    - elasticsearch
  networks:
    - backend
```

## 设置抽样比例

默认情况下，Zipkin 仅将 10% 的事务写入Zipkin服务器。

可以通过设置 <mark style="color:blue;">**management.tracing.sampling.probability**</mark> 属性来调整这个比例，该属性接受0 到 1 之间的值。

```yaml
management:
  tracing:
    sampling:
      probability: 1.0
```

## 添加自定义span

可以通过启动一个观察来创建自己的span。为此，需要将 ObservationRegistry 注入到您的组件中。

以下示例为名为 readLicensingDataFromRedis 的许可服务创建了一个自定义跨度：

<pre class="language-java" data-overflow="wrap" data-line-numbers><code class="lang-java"><strong>@Autowired
</strong><strong>private ObservationRegistry observationRegistry;
</strong>
private Organization checkRedisCache(String organizationId) {
    Observation newSpan = Observation
            .createNotStarted("readLicensingDataFromRedis", observationRegistry)
            .lowCardinalityKeyValue("peer.service", "redis")
            .start();
    try {
        return redisRepository
                .findById(organizationId)
                .orElse(null);
    } catch (Exception ex) {
        log.error("Error encountered while trying to retrieve organization{} check Redis Cache. Exception {}",
                organizationId, ex);
        return null;
    } finally {
        newSpan.stop();
    }
}
</code></pre>
