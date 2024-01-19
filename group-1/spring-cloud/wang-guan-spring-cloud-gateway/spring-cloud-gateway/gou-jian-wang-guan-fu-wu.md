# 构建网关服务

## 1. 创建项目

使用Spring Initializer，选择**Reactive Gateway**、**Eureka Client**、**Spring Boot Actuator**和**Config Client**，生成项目的pom.xml的核心文件内容如下：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

## 2. 添加配置属性

在**/src/main/resources**文件夹中创建<mark style="color:blue;">**application.yml**</mark>文件：

```yaml
spring:
    profiles:
        active: dev
    application:
        name: gateway-server    
    cloud:
        config:
            uri: http://192.168.10.110:8071/
            fail-fast: true
    config:
        import: 'optional:configserver:'
```

## 3. 集成Eureka

Spring Cloud Gateway可以与Netflix Eureka Discovery服务集成。要添加新的网关服务，第一步是在Spring配置服务器存储库中为此服务创建一个配置文件。对于这个示例，我们创建了gateway-server.yml文件。

接下来，我们将Eureka配置数据添加到刚刚创建的配置文件中：

```yaml
server:
  port: 8072
eureka:
  instance:
    preferIpAddress: true
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://192.168.10.110:8070/eureka/
```

通过这样的配置，Spring Cloud Gateway将能够注册到Eureka服务注册中心，并发现其他服务的实例。
