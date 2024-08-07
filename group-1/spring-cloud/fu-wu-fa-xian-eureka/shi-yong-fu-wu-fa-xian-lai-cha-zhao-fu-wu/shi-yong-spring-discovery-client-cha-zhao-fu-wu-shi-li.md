# 使用 Spring Discovery Client 查找服务实例

<mark style="color:blue;">**Spring Discovery Client**</mark> 提供了对 Load Balancer 及其注册服务的<mark style="color:blue;">**最低层次的**</mark>访问。使用 Discovery Client，可以查询所有服务以及这些服务对应的 URL。

## 1. 使用 **@EnableDiscoveryClient** 注解标注引导类

```java
package com.study.cloudlearning;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.context.config.annotation.RefreshScope;

@RefreshScope
@EnableDiscoveryClient
@SpringBootApplication
public class CloudLearningApplication {

    public static void main(String[] args) {
        SpringApplication.run(CloudLearningApplication.class, args);
    }
}
```

<mark style="color:blue;">**@EnableDiscoveryClient**</mark> 注解是 Spring Cloud 的触发器，其作用是使应用程序能够使用 Discovery Client 和 Spring Cloud LoadBalancer 库。&#x20;

## 2. 注入 DiscoveryClient Bean 对象

```java
private DiscoveryClient discoveryClient;

@Autowired
public void setDiscoveryClient(DiscoveryClient discoveryClient) {
    this.discoveryClient = discoveryClient;
}
```

DiscoveryClient 的全限定名称是：<mark style="color:blue;">**org.springframework.cloud.client.discovery.DiscoveryClient**</mark>

## 3. 通过 DiscoveryClient 查找指定服务

{% code overflow="wrap" %}
```java
List<ServiceInstance> organizationInstances = discoveryClient
                .getInstances("organization-service");
```
{% endcode %}

## 4. 使用 RestTemplate 访问服务

```java
RestTemplate restTemplate = new RestTemplate();

Organization organization = organizationInstances.stream()
                .findFirst()
                .map(ServiceInstance::getUri)
                .map(uri -> uri + "/v1/organization/{organizationId}")
                .map(serviceUri -> restTemplate.getForObject(
                        serviceUri,
                        Organization.class,
                        organizationId)
                ).orElse(null);
```
