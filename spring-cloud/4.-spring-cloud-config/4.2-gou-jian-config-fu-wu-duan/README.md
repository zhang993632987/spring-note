# 4.2 构建Config服务端

Spring Cloud Config服务器端是**基于REST**的应用程序，它建立在Spring Boot之上。Config 服务器端不必是一个独立的服务器端，相反，你可以选择将它**嵌入现有的Spring Boot应用程序**中，也可以开启新的Spring Boot项目然后嵌入Config服务器端。<mark style="color:orange;">**最佳实践是保持分离。**</mark>

## 1. 创建项目

使用Spring Initializer，选择**Spring Boot Actuator**和**Config Server**，生成项目的pom.xml的核心文件内容如下：

```xml
<properties>
    <java.version>11</java.version>
    <spring-cloud.version>2021.0.8</spring-cloud.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

{% hint style="warning" %}
需要注意的是，Spring Boot是独立于Spring Cloud发布列车发布的。因此，<mark style="color:orange;">**Spring Boot的不同版本可能与Spring Cloud的发布不兼容。**</mark>
{% endhint %}

## 2. 添加配置属性

在**/src/main/resources**文件夹中创建<mark style="color:blue;">**application.yml**</mark>文件。这个文件告诉Spring Cloud Config服务监听什么端口、应用程序名称、应用程序配置文件以及我们将用于存储配置数据的位置。内容如下：

```yaml
server:
  port: 8071
spring:
  application:
    name: config-server
```

## 3. 在引导类上增加@EnableConfigServer注解

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
