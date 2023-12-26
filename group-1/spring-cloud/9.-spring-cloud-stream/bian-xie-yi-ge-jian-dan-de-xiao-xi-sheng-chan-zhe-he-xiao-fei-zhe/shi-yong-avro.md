# 使用avro

## 部署schema registry

```yaml
schema-registry:
  image: docker.io/bitnami/schema-registry:7.5
  restart: always
  ports:
    - '8074:8081'
  depends_on:
    - kafka
  environment:
    - SCHEMA_REGISTRY_LISTENERS=http://0.0.0.0:8081
    - SCHEMA_REGISTRY_KAFKA_BROKERS=PLAINTEXT://kafka:9092
```

## 添加依赖

<details>

<summary><mark style="color:purple;"><strong>properties</strong></mark></summary>

```xml
<confluent.version>5.2.0</confluent.version>
<avro.version>1.11.3</avro.version>
```

</details>

<details>

<summary><mark style="color:purple;"><strong>dependencise</strong></mark></summary>

<pre class="language-xml"><code class="lang-xml"><strong>&#x3C;dependency>
</strong>    &#x3C;groupId>org.apache.avro&#x3C;/groupId>
    &#x3C;artifactId>avro&#x3C;/artifactId>
    &#x3C;version>${avro.version}&#x3C;/version>
&#x3C;/dependency>
&#x3C;dependency>
    &#x3C;groupId>io.confluent&#x3C;/groupId>
    &#x3C;artifactId>kafka-streams-avro-serde&#x3C;/artifactId>
    &#x3C;version>${confluent.version}&#x3C;/version>
&#x3C;/dependency>
&#x3C;dependency>
    &#x3C;groupId>io.confluent&#x3C;/groupId>
    &#x3C;artifactId>kafka-avro-serializer&#x3C;/artifactId>
    &#x3C;version>${confluent.version}&#x3C;/version>
    &#x3C;exclusions>
        &#x3C;exclusion>
            &#x3C;groupId>org.slf4j&#x3C;/groupId>
            &#x3C;artifactId>slf4j-api&#x3C;/artifactId>
        &#x3C;/exclusion>
        &#x3C;exclusion>
            &#x3C;groupId>org.slf4j&#x3C;/groupId>
            &#x3C;artifactId>slf4j-log4j12&#x3C;/artifactId>
        &#x3C;/exclusion>
    &#x3C;/exclusions>
&#x3C;/dependency>
</code></pre>

</details>

<details>

<summary><mark style="color:purple;">repositories</mark></summary>

```xml
<repository>
    <id>confluent</id>
    <url>https://packages.confluent.io/maven/</url>
</repository>
```

</details>

<details>

<summary><mark style="color:purple;">plugins</mark></summary>

```
<plugin>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro-maven-plugin</artifactId>
    <version>${avro.version}</version>
    <configuration>
        <sourceDirectory>src/main/resources/avro</sourceDirectory>
        <outputDirectory>src/main/java</outputDirectory>
    </configuration>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>schema</goal>
                <goal>protocol</goal>
                <goal>idl-protocol</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

</details>

{% hint style="warning" %}
## <mark style="color:orange;">注意</mark>

为了让 pom.xml 中配置的仓库地址能够生效，需要修改 Maven 的 settings 文件，让镜像不要代理 confluent 仓库，如下所示：

```xml
<mirror>
  <id>aliyunmaven</id>
  <mirrorOf>*,!confluent</mirrorOf>
  <name>阿里云公共仓库</name>
  <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```
{% endhint %}

### 使用avro插件生成类

<details>

<summary><mark style="color:purple;">avsc 文件</mark></summary>

```json
{
  "namespace": "com.study.organization.model",
  "type": "record",
  "name": "OrganizationChangeModel",
  "fields": [
    {
      "name": "typeName",
      "type": "string"
    },
    {
      "name": "action",
      "type": "string"
    },
    {
      "name": "organizationId",
      "type": "string"
    },
    {
      "name": "correlationId",
      "type": ["null", "string"],
      "default": null
    }
  ]
}
```

</details>

执行命令：

```properties
mvn avro:schema
```

## 配置 channel

### 生产者

```yaml
spring:
  cloud:
    stream:
      function:
        definition: send
        bindings:
          send-out-0: send-org
      bindings:
        send-out-0:
          destination: orgChangeTopic
          group: organization
          producer:
            useNativeEncoding: true
      kafka:
        bindings:
          send-out-0:
            producer:
              configuration:
                schema.registry.url: http://192.168.10.110:8074
                value.serializer: io.confluent.kafka.serializers.KafkaAvroSerializer
        binder:
          brokers: 192.168.10.110:9094
          requiredAcks: all
```

### 消费者

```yaml
spring:
  cloud:
    stream:
      function:
        definition: consume
      bindings:
        consume-in-0:
          destination: orgChangeTopic
          group: license
          consumer:
            useNativeEncoding: true
      kafka:
        bindings:
          consume-in-0:
            consumer:
              configuration:
                schema.registry.url: http://192.168.10.110:8074
                value.deserializer: io.confluent.kafka.serializers.KafkaAvroDeserializer
        binder:
          brokers: 192.168.10.110:9094
          requiredAcks: all
```
