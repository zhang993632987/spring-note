# Distributed tracing with Zipkin

Distributed tracing involves providing a visual picture of how a transaction flows across our different microservices. Distributed tracing tools also give a rough approximation of individual microservice response times.

Zipkin (http://zipkin.io/) is a distributed tracing platform that allows us to trace transactions across multiple service invocations. It lets us graphically see the amount of time a transaction takes and breaks down the time spent in each microservice involved in the call. Zipkin is an invaluable tool for identifying performance issues in a microservices architecture. Setting up Spring Cloud Sleuth and Zipkin involves

* Adding Spring Cloud Sleuth and Zipkin JAR files to the services that capture trace data
* Configuring a Spring property in each service to point to the Zipkin server that will collect the trace data
* Installing and configuring a Zipkin server to collect the data
* Defining the sampling strategy each client will use to send tracing information to Zipkin

## Setting up the Spring Cloud Sleuth and Zipkin dependencies

```xml
<dependency>
    <groupId>io.zipkin.reporter2</groupId>
    <artifactId>zipkin-reporter-brave</artifactId>
</dependency>
```

## Configuring the services to point to Zipkin

With the JAR files in place, we need to configure each service that wants to communicate with Zipkin. We’ll do this by setting a Spring property that defines the URL used to communicate with Zipkin. The property that needs to be set is the management.zipkin.tracing.endpoint property:&#x20;

```yaml
management:
  zipkin:
    tracing:
      endpoint: http://kubernetes:9411/api/v2/spans
```

> Zipkin has the ability to send its tracing data to a Zipkin server via RabbitMQ or Kafka. From a functionality perspective, there’s no difference in Zipkin’s behavior if we use HTTP, RabbitMQ, or Kafka. With HTTP tracing, Zipkin uses an asynchronous thread to send performance data. The main advantage of using RabbitMQ or Kafka to collect tracing data is that if our Zipkin server is down, tracing messages sent to Zipkin are “enqueued” until Zipkin can pick up the data.

## Configuring a Zipkin server

To set up Zipkin, we’ll add the following registry entry to our compose.yml file located in the Docker folder for the project:

To run a Zipkin server, little configuration is needed. One of the few things we need to configure when we run Zipkin is the backend data store that Zipkin uses to store the tracing data. Zipkin supports four different backend data stores:

* In-memory data
* MySQL
* Cassandra
* Elasticsearch

By default, Zipkin uses an in-memory data store for storage.

For this example, we’ll show you how to use Elasticsearch as a data store because we’ve already configured Elasticsearch. The only additional settings we need to add are the STORAGE\_TYPE and ES\_HOSTS variables in the environment section of our configuration file. The following code shows the complete Docker Compose registry:

```
zipkin: 
    image: openzipkin/zipkin 
    container_name: zipkin
    depends_on: 
      - elasticsearch
    environment: 
      - STORAGE_TYPE=elasticsearch
      - "ES_HOSTS=elasticsearch:9300"
    ports:
      - "9411:9411"
    networks:
      backend:
        aliases:
          - "zipkin"
```

## Setting tracing levels

By default, Zipkin only writes 10% of all transactions to the Zipkin server. The default value ensures that Zipkin will not overwhelm our logging and analysis infrastructure.

The transaction sampling can be controlled by setting a Spring property on each of the services sending data to Zipkin: the management.tracing.sampling.probability property. This property takes a value between 0 and 1 as follows:

* A value of 0 means Spring Cloud Sleuth doesn’t send Zipkin any transactions.
* A value of .5 means Spring Cloud Sleuth sends 50% of all transactions.
* A value of 1 means Spring Cloud Sleuth sends all transactions (100%).

```yaml
management:
  tracing:
    sampling:
      probability: 1.0
```

### Adding custom spans

You can create your own spans by starting an observation. For this, inject ObservationRegistry into your component. The following listing creates a custom span for the licensing service that’s called readLicensingDataFromRedis.

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
