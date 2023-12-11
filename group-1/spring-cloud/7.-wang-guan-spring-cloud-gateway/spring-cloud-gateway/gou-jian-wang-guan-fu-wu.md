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

在**/src/main/resources**文件夹中创建<mark style="color:blue;">**application.yml**</mark>文件。这个文件告诉Eureka服务监听什么端口、应用程序名称、Config Server地址。

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

The Spring Cloud Gateway can integrate with the Netflix Eureka Discovery service.To add a new Gateway service, the first step is to create a configuration file for this service in the Spring Configuration Server repository. For this example, we’ve created the gateway-server.yml file.

Next, we will add the Eureka configuration data into the configuration file we just created. The following listing shows how.

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
