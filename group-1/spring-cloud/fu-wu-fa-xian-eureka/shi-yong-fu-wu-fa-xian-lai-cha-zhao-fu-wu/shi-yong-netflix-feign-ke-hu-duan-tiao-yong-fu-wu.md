# 使用Netflix Feign客户端调用服务

Netflix的Feign客户端库是启用Load Balancer的RestTemplate类的替代方案。**Feign库采用不同的方法来调用REST服务。**使用这种方法，开发人员首先定义一个Java接口，然后添加Spring Cloud注解，以映射将要调用的服务。Spring Cloud框架将动态生成一个代理类，用于调用目标REST服务。<mark style="color:blue;">**除了接口定义，开发人员不需要编写其他调用服务的代码。**</mark>

## **1. 添加依赖**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

## 2. 使用**@EnableFeignClients**注解标注引导类

```java
package com.study.cloudlearning;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@RefreshScope
@EnableFeignClients
@SpringBootApplication
public class CloudLearningApplication {

    public static void main(String[] args) {
        SpringApplication.run(CloudLearningApplication.class, args);
    }

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

## 3.定义接口

```java
package com.study.cloudlearning.service.client;

import com.study.cloudlearning.entity.Organization;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

/**
 * @author Zhang B H
 * @create 2023-10-30 12:40
 */

@FeignClient("organization-service")
public interface OrganizationFeignClient {

    @GetMapping("/v1/organization/{organizationId}")
    Organization getOrganization(
            @PathVariable("organizationId") String organizationId);
}
```

上述代码中，使用了@<mark style="color:blue;">**FeignClient**</mark>注解，并将我们想让这个接口代表的服务的<mark style="color:blue;">**应用程序ID**</mark>传递给它。接下来，在这个接口中定义一个getOrganization()方法，客户端可以调用该方法来调用组织服务。

定义getOrganization()方法的方式看起来就像在Spring控制器类中公开一个端点的方式一样。

* 首先，为getOrganization()方法定义一个@RequestMapping注解，该注解映射HTTP动词以及将在 organization 服务调用中公开的端点。
* 其次，使用@PathVariable注解将URL传入的组织ID映射到方法调用上的organizationId参数。
* 调用组织服务的返回值将被自动映射到Organization类，这个类被定义为getOrganization()方法的返回值。

<mark style="color:blue;">**要使用OrganizationFeignClient类，我们需要做的只是自动装配并使用它。Feign客户端代码将为我们处理所有编码。**</mark>

> ## <mark style="color:blue;">**错误处理**</mark>
>
> <mark style="color:blue;">**通过Feign客户端，任何被调用的服务返回的HTTP状态码4xx \~ 5xx都将映射为FeignException。**</mark>
>
> FeignException包含可以被解析为特定错误消息的JSON体。
>
> Feign提供了编写错误解码器类的功能，该类可以将错误映射回自定义的Exception类。
