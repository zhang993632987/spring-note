# 构建 Config 服务端

Spring Cloud Config 服务器端是**基于 REST** 的应用程序，它建立在 Spring Boot 之上。

> Config 服务器端不必是一个独立的服务器端：可以选择将它**嵌入现有的 Spring Boot 应用程序**中，也可以开启新的 Spring Boot 项目然后嵌入Config 服务器端。
>
> <mark style="color:orange;">**最佳实践是保持分离。**</mark>

## 1. 创建项目

使用 Spring Initializer，选择 **Spring Boot Actuator** 和 **Config Server**，生成项目的 pom.xml 的核心内容如下：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

## 2. 添加配置属性

在 **/src/main/resources** 文件夹中创建 <mark style="color:blue;">**application.yml**</mark> 文件，内容如下：

```yaml
server:
  port: 8071
spring:
  application:
    name: config-server
```

## 3. 在引导类上增加 @EnableConfigServer 注解

```java
package com.study.config.server;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@EnableConfigServer
@SpringBootApplication
public class ConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }

}
```
