# Configuring Apache Kafka and Redis in Docker

To achieve this, let’s start by adding the code shown in the following listing to our docker-compose.yml file.

```yaml
zookeeper:
    image: wurstmeister/zookeeper:latest
    ports:
      - 2181:2181
    networks:
      backend:
        aliases:
          - "zookeeper"
  kafkaserver:
    image: wurstmeister/kafka:latest
    ports:
      - 9092:9092
    environment:
    - KAFKA_ADVERTISED_HOST_NAME=kafka
    - KAFKA_ADVERTISED_PORT=9092
    - KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181
    - KAFKA_CREATE_TOPICS=dresses:1:1,ratings:1:1
  volumes:
    - "/var/run/docker.sock:/var/run/docker.sock"
  depends_on:
    - zookeeper
  networks:
    backend:
      aliases:
        - "kafka"
redisserver:
  image: redis:alpine
  ports:
    - 6379:6379
  networks:
    backend:
      aliases:
        - "redis"
```
