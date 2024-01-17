# Spring Boot客户端集成Config服务

## 1. 增加Maven依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

## 2. application.yml配置文件

在定义了Maven依赖项后，我们需要告知微服务在哪里与Spring Cloud Config服务器端进行联系。

```yaml
spring:
  profiles:
    active: dev
  application:
    name: cloud-learning
  cloud:
    config:
      uri: http://localhost:8071/
  config:
    import: 'optional:configserver:'
```

* **spring.application.name**是应用程序的名称，必须直接映射到Spring Cloud Config服务器端中的配置文件的名称。
* **spring.profiles.active**用于告诉Spring Boot应用程序应该运行哪个profile。
* **spring.cloud.config.uri**是Config服务器端端点的位置。
* **spring.config.import=**optional:configserver:

> 如果要覆盖这些默认值，可以通过将客户端服务项目编译并打包到JAR文件，然后使用<mark style="color:blue;">**-D**</mark>系统属性来运行这个JAR文件来实现：
>
> ```bash
> java -Dspring.cloud.config.uri=http://localhost:8071 \
>      -Dspring.profiles.active=dev \
>      -jar target/cloud-learning-0.0.1-SNAPSHOT.jar
> ```
