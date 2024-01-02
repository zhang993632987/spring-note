# 日志聚合

要设置 ELK 需要采取以下步骤：

1. 在服务中配置Logback
2. 在Docker容器中定义和运行 ELK Stack 应用程序
3. 配置 Kibana
4. 使用 Trace ID进行查询

## 配置 Logback&#x20;

### 添加 LOGSTASH ENCODER

首先，需要将 **logstash-logback-encoder** 依赖添加到许可（licensing）、组织（organization）和网关（gateway）服务的 pom.xml 文件中。

```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```

### 创建 LOGSTASH TCP APPENDER

在添加了依赖后，需要告诉微服务以JSON格式的形式与Logstash进行通信。默认情况下，Logback以纯文本形式生成应用程序日志，但为了使用Elasticsearch索引，需要确保以JSON格式发送日志数据。有三种方法可以实现这一点：

* 使用net.logstash.logback.encoder.LogstashEncoder类
*   使用net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder类

    通过LoggingEventCompositeJsonEncoder，可以添加新的模式或字段，禁用默认provider等
* 使用Logstash解析纯文本日志数据

> 在以下示例中，将使用LogstashEncoder。之所以选择这个类是因为它实现起来最简单、最快速，并且在这个示例中，并不需要向记录器添加额外的字段。

为了配置这个编码器，创建一个名为**logback-spring.xml** 的Logback配置文件。这个配置文件应该位于服务资源文件夹（resources）中。

{% code overflow="wrap" %}
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml"/>
    <springProperty scope="context" name="application_name"
                    source="spring.application.name"/>
    <appender name="logstash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>kubernetes:5000</destination>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>
    <root level="INFO">
        <appender-ref ref="logstash"/>
        <appender-ref ref="CONSOLE"/>
    </root>
    <logger name="org.springframework" level="INFO"/>
    <logger name="com.study" level="DEBUG"/>
</configuration>
```
{% endcode %}

## 在Docker容器中定义和运行 ELK Stack 应用程序

> Logstash管道有两个必需的和一个可选的元素。
>
> * 必需的元素是输入（inputs）和输出（outputs）：
>   * 输入允许Logstash读取特定事件源的事件。Logstash支持各种输入插件，如GitHub、Http、TCP、Kafka等。
>   * 输出负责将事件数据发送到特定目的地。Elastic支持各种输出插件，如CSV、Elasticsearch、email、file、MongoDB、Redis、stdout等。
> * 可选的元素是过滤器插件。这些过滤器负责对事件执行中间处理，如翻译、添加新信息、解析日期、截断字段等。



<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

在这个例子中，将使用之前配置的 Logback TCP appender 作为输入插件，而输出插件将使用Elasticsearch引擎：

```nginx
input {    
  tcp {
    port => 5000    
    codec => json_lines
  }
}
filter {
  mutate {
    add_tag => [ "self-learning" ]   
  }
}
output {    
  elasticsearch {
    hosts => "elasticsearch:9200"   
  }
}
```

在上面的示例中，可以看到五个关键元素：

* 第一个是输入部分，这个部分指定了使用tcp插件来消费日志数据。
* 接下来是端口号5000，这是Logstash监听的端口。&#x20;
* 第三个元素是可选的，对应于过滤器；对于这个特定的场景，添加了一个mutate过滤器。这个过滤器向事件添加了一个名为 self-learning 的标签。在实际应用中，可能是服务所运行的环境的一个可能的标签。
* 最后，第四和第五个元素指定了Logstash服务的输出插件，并将处理过的数据发送到运行在端口号9200上的Elasticsearch服务。&#x20;

```yaml
services:
  elasticsearch:
    image: elasticsearch:7.5.2
    restart: always
    ports:
      - '9200:9200'
    environment:
      - discovery.type=single-node
    volumes:
      - '/bitnami/elasticsearch/data'
    networks:
      - backend  
  kibana:
    image: kibana:7.5.2
    restart: always
    ports:
      - '5601:5601'
    environment:
      - ELASTICSEARCH_HOSTS=["http://elasticsearch:9200"]
    depends_on:
      - elasticsearch
    networks:
      - backend
  logstash:
    extends:
      file: logstash/compose.yaml
      service: logstash
    depends_on:
      - elasticsearch
    networks:
      - backend
```

<details>

<summary><mark style="color:purple;">logstash</mark></summary>

{% code title="logstash/Dockerfile" %}
```dockerfile
FROM logstash:7.5.2

RUN rm -f /usr/share/logstash/pipeline/logstash.conf

ADD pipeline/ /usr/share/logstash/pipeline/

ADD config/ /usr/share/logstash/config/
```
{% endcode %}

{% code title="logstash/compose.yaml" %}
```yaml
services:
  logstash:
    image: logstash:v1
    build:
      context: .
    restart: always
    ports:
      - 5000:5000
```
{% endcode %}

</details>

## 配置 Kibana

要访问Kibana，请在Web浏览器中打开以下链接：http://localhost:5601/。

第一次访问Kibana时，将显示一个欢迎页面。该页面包含两个选项：

* 第一个选项允许我们使用一些示例数据进行操作
* 而第二个选项允许我们探索从我们的服务生成的数据

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

为了探索我们的数据，点击“Explore on My Own”链接。一旦点击，我们将看到一个“Add Data”页面。在这个页面上，我们需要点击页面左侧的“Discover”图标。

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

接下来，我们必须创建一个索引模式。Kibana使用一组索引模式从Elasticsearch引擎中检索数据。

索引模式负责告诉Kibana我们想要探索哪些Elasticsearch索引。例如，我们将创建一个索引模式，指示我们想要从Elasticsearch检索所有Logstash信息。要创建我们的索引模式，请点击页面左侧Kibana部分下的“Index Patterns”链接。

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

在上图中的“Create Index Pattern”页面上，我们可以看到Logstash已经创建了一个索引。为了完成索引的设置，我们必须为该索引指定一个索引模式。要创建它，我们需要编写索引模式logstash-\*，然后点击“Next Step”按钮。

然后，我们将指定一个时间过滤器。为此，我们需要在“Time Filter Field Name”下拉列表中选择@timestamp选项，然后点击“Create Index Pattern”按钮。

<figure><img src="../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

现在我们可以开始向我们的服务发出请求，并在Kibana中查看实时日志了。

## 在 Kibana 上查询事务日志

要查询与单个事务相关的所有日志条目，我们需要获取一个跟踪ID并在Kibana的Discover界面上查询它。默认情况下，Kibana使用Kibana查询语言（Kibana Query Language，KQL），这是一种简化的查询语法。

<figure><img src="../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>
