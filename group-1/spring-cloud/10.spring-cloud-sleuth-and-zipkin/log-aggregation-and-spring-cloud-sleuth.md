# Log aggregation and Spring Cloud Sleuth

To setup ELK to work with our environment, we need to take the following actions:

1. Configure Logback in our services
2. Define and run the ELK Stack applications in Docker containers
3. Configure Kibana
4. Test the implementation by issuing queries based on the correlation IDs from Spring Cloud Sleuth

## Configuring Logback in our services

### ADDING THE LOGSTASH ENCODER

To begin, we need to add the logstash-logback-encoder dependency to the pom.xml file of our licensing, organization, and gateway services.

```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```

### CREATING THE LOGSTASH TCP APPENDER

Once the dependency is added to each service, we need to tell the licensing service that it needs to communicate with Logstash to send the applications logs, formatted as JSON. (Logback, by default, produces the application logs in plain text, but to use Elasticsearch indexes, we need to make sure we send the log data in a JSON format.) There are three ways to accomplish this:

* Using the net.logstash.logback.encoder.LogstashEncoder class
* Using the net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder class
* Parsing the plain-text log data with Logstash

For this example, we will use LogstashEncoder. We’ve chosen this class because it is the easiest and fastest to implement and because, in this example, we don’t need to add additional fields to the logger. With LoggingEventCompositeJsonEncoder, we can add new patterns or fields, disable default providers, and more. With the third option, we can delegate parsing entirely to the Logstash using a JSON filter. All three options are good, but we suggest using the LoggingEventCompositeJsonEncoder when you have to add or remove default configurations.

To configure this encoder, we’ll create a Logback configuration file called logback-spring.xml. This configuration file should be located in the service resources folder.

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

You can also configure the log data shown in figure 11.5 by using the LoggingEventCompositeJsonEncoder. Using this composite encoder, we can disable all the providers that were added by default to our configuration, add new patterns to display custom or existent MDC fields, and more.&#x20;

## Defining and running ELK Stack applications in Docker

To set up our ELK Stack containers, we need to follow two simple steps. The first is to create the Logstash configuration file, and the second is to define the ELK Stack applications in our Docker configuration. Before we start creating our configuration, though, it’s important to note that the Logstash pipeline has two required and one optional element. The required elements are the inputs and outputs:

* The input enables a specific source of events to be read by Logstash. Logstash supports a variety of input plugins such as GitHub, Http, TCP, Kafka, and others.
* The output is in charge of sending the event data to a particular destination. Elastic supports a variety of plugins such as CSV, Elasticsearch, email, file, MongoDB, Redis, stdout, and others.

The optional element in the Logstash configuration is the filter plugins. These filters are in charge of performing intermediary processing on an event such as translations, adding new information, parsing dates, truncating fields, and so forth.

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

For this example, we’ll use as the input plugin the Logback TCP appender that we previously configured and as the output plugin the Elasticsearch engine. The following listing shows the /docker/config/logstash.conf file.

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

In listing 11.3, we can see five essential elements. The first is the input section. In this section, we specify the tcp plugin for consuming the log data. Next is the port number 5000; this is the port that we’ll specify for Logstash later in the docker-compose.yml file.

The third element is optional and corresponds to the filters; for this particular scenario, we added a mutate filter. This filter adds a manningPublications tag to the events. A realworld scenario of a possible tag for your services might be the environment where the application runs. Finally, the fourth and fifth elements specify the output plugin for our Logstash service and send the processed data to the Elasticsearch service running on port 9200.

Now that we have the Logstash configuration, let’s add the three ELK Docker entries to the docker-compose.yml file.

```yaml
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

<summary>logstash</summary>

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

## Configuring Kibana

To access Kibana, open a web browser to the following link: http://localhost:5601/. The first time we access Kibana, a Welcome page is displayed. This page shows two options. The first allows us to play with some sample data, and the second lets us explore the generated data from our services.

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

To explore our data, let’s click the Explore on My Own link. Once clicked, we will see an Add Data page. On this page, we need to click the Discover icon on the left side of the page.

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

In order to continue, we must create an index pattern. Kibana uses a set of index patterns to retrieve the data from an Elasticsearch engine. The index pattern is incharge of telling Kibana which Elasticsearch indexes we want to explore. For example, in our case, we will create an index pattern indicating that we want to retrieve all of the Logstash information from Elasticsearch. To create our index pattern, click the Index Patterns link under the Kibana section at the left of the page.

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

On the Create Index Pattern page in figure 11.9, we can see that Logstash has already created an index as a first step. However, this index is not ready to use yet. To finish setting up the index, we must specify an index pattern for that index. To create it, we need to write the index pattern, logstash-\*, and click the Next Step button.

For step 2, we’ll specify a time filter. To do this, we need to select the @timestamp option under the Time Filter Field Name drop-down list and then click the Create Index Pattern button.

<figure><img src="../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

We can now start making requests to our services to see the real-time logs in Kibana.

## Searching for Spring Cloud Sleuth trace IDs in Kibana

To query for all the log entries related to a single transaction, we need to take a trace ID and query it on Kibana’s Discover screen(figure 11.12). By default, Kibana uses the Kibana Query Language (KQL), which is a simplified query syntax. When writing our query, you’ll see that Kibana also provides a guide and autocomplete option to simplify the process of creating custom queries.

<figure><img src="../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>
