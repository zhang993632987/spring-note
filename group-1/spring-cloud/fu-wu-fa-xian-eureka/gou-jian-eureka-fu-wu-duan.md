# 构建 Eureka 服务端

## 1. 创建项目

使用 Spring Initializer，选择 **Eureka Server**、**Spring Boot Actuator** 和 **Config Client**，生成项目的 pom.xml 的核心内容如下：

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
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

## 2. 添加配置属性

在 **/src/main/resources** 文件夹中创建 <mark style="color:blue;">**application.yml**</mark> 文件：

```yaml
spring:
  application:
    name: eureka-server
  cloud:
    config:
      uri: http://localhost:8071/
      request-connect-timeout: 10000
      request-read-timeout: 10000
    loadbalancer:
      enabled: false
  config:
    import: "optional:configserver:"
```

## 3. 在引导类上增加 @EnableEurekaServer 注解

```java
package com.study.eureka.server;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}
```

## 4. 在 Config Server 上添加配置

```yaml
server:
  port: 8070
eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
  server:
    wait-time-in-ms-when-sync-empty: 5
```

* <mark style="color:blue;">**server.port：**</mark>设置默认端口。
* <mark style="color:blue;">**eureka.instance.hostname：**</mark>设置 Eureka 服务的 Eureka 实例主机名。
* <mark style="color:blue;">**eureka.client.registerWithEureka：**</mark>告诉 Eureka 服务器端别在应用程序启动时注册到 Eureka。
* <mark style="color:blue;">**eureka.client.fetchRegistry：**</mark>当设置为 false 时，告诉 Eureka 服务，当它启动时，不需要在本地缓存其注册表信息。
*   <mark style="color:blue;">**eureka.client.serviceUrl.defaultZone：**</mark>连接的 eureka 服务端的地址。

    > <mark style="color:orange;">**默认情况下，每一个 Eureka 服务同时也是一个 Eureka 客户，因此必须为它配置一个指定对等 Eureka 服务的 URL 地址。**</mark>
*   <mark style="color:blue;">**eureka.server.waitTimeInMsWhenSyncEmpty：**</mark>设置服务器端在开始接受请求之前的等待时间（以毫秒为单位）。

    > 当你本地测试服务时，应该使用这一行属性，因为 Eureka 不会马上通告任何通过它注册的服务。<mark style="color:blue;">**默认情况下，它会等待 5 分钟，让所有的服务都有机会在通告它们之前通过它来注册。**</mark>进行本地测试时使用这行属性，将有助于加快 Eureka 服务启动和显示通过它注册服务所需的时间。
