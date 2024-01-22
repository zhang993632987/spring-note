# 配置Apache Kafka和Redis

<details>

<summary><mark style="color:purple;">compose.yaml</mark></summary>

<pre class="language-yaml" data-overflow="wrap"><code class="lang-yaml">services:
  kafka:
    image: docker.io/bitnami/kafka:3.6
<strong>    ports:
</strong>      - "9092:9092"
      - "9094:9094"
    volumes:
      - "kafka_data:/bitnami"
    environment:
      # KRaft settings
      - KAFKA_CFG_NODE_ID=0
      - KAFKA_CFG_PROCESS_ROLES=controller,broker
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@kafka:9093
      # Listeners
      - KAFKA_CFG_LISTENERS=BROKER://:9092,CONTROLLER://:9093,CLIENT://:9094
      - KAFKA_CFG_ADVERTISED_LISTENERS=BROKER://:9092,CLIENT://:9094
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,CLIENT:PLAINTEXT,BROKER:PLAINTEXT
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - KAFKA_CFG_INTER_BROKER_LISTENER_NAME=BROKER
  redis:
    image: docker.io/bitnami/redis:7.2
    environment:
      # ALLOW_EMPTY_PASSWORD is recommended only for development.
      - ALLOW_EMPTY_PASSWORD=yes
      - REDIS_DISABLE_COMMANDS=FLUSHDB,FLUSHALL
    ports:
      - '6379:6379'
    volumes:
      - 'redis_data:/bitnami/redis/data' 
volumes:
  kafka_data:
    driver: local
  redis_data:
    driver: local
</code></pre>

</details>

{% hint style="warning" %}
## <mark style="color:orange;">注意</mark>

kafka客户端会向broker请求kafka broker的所有元数据，**bootstrap.servers实际上是引导地址，而不是客户端真正建立长链接的地址。**也就是说，**客户端会根据引导地址去broker询问集群的所有broker信息，拿到返回的broker服务信息之后，再向指定的broker发起链接请求。**

```properties
advertised.listeners=BROKER://:9092,CLIENT://:9094
```

根据上述 advertised.listeners 的配置，返回给客户端的元信息中的地址也是 hostname:9094，当客户端准备根据这个hostname建立长连接请求数据的时候，发现并解析不了该 hostname，从而导致出现以下异常：

```log
java.net.UnknownHostException: 1350d4bf3a81
	at java.base/java.net.InetAddress$CachedAddresses.get(InetAddress.java:797) ~[na:na]
	at java.base/java.net.InetAddress.getAllByName0(InetAddress.java:1505) ~[na:na]
	at java.base/java.net.InetAddress.getAllByName(InetAddress.java:1364) ~[na:na]
	at java.base/java.net.InetAddress.getAllByName(InetAddress.java:1298) ~[na:na]
	at org.apache.kafka.clients.DefaultHostResolver.resolve(DefaultHostResolver.java:27) ~[kafka-clients-3.1.2.jar:na]
	at org.apache.kafka.clients.ClientUtils.resolve(ClientUtils.java:110) ~[kafka-clients-3.1.2.jar:na]
	at org.apache.kafka.clients.ClusterConnectionStates$NodeConnectionState.currentAddress(ClusterConnectionStates.java:511) ~[kafka-clients-3.1.2.jar:na]
	at org.apache.kafka.clients.ClusterConnectionStates$NodeConnectionState.access$200(ClusterConnectionStates.java:468) ~[kafka-clients-3.1.2.jar:na]
	at org.apache.kafka.clients.ClusterConnectionStates.currentAddress(ClusterConnectionStates.java:173) ~[kafka-clients-3.1.2.jar:na]
	at org.apache.kafka.clients.NetworkClient.initiateConnect(NetworkClient.java:988) ~[kafka-clients-3.1.2.jar:na]
	at org.apache.kafka.clients.NetworkClient.ready(NetworkClient.java:301) ~[kafka-clients-3.1.2.jar:na]
	at org.apache.kafka.clients.admin.KafkaAdminClient$AdminClientRunnable.sendEligibleCalls(KafkaAdminClient.java:1128) ~[kafka-clients-3.1.2.jar:na]
	at org.apache.kafka.clients.admin.KafkaAdminClient$AdminClientRunnable.processRequests(KafkaAdminClient.java:1388) ~[kafka-clients-3.1.2.jar:na]
	at org.apache.kafka.clients.admin.KafkaAdminClient$AdminClientRunnable.run(KafkaAdminClient.java:1331) ~[kafka-clients-3.1.2.jar:na]
	at java.base/java.lang.Thread.run(Thread.java:834) ~[na:na]

```

**为了解决上述异常，需要在客户端的 etc/hosts 文件中添加该hostname的ip映射之后便会根据域名解析找到ip建立连接。**
{% endhint %}
