# 使用Spring Discovery Client查找服务实例

<mark style="color:blue;">**Spring Discovery Client**</mark>提供了对Load Balancer及其注册服务的<mark style="color:blue;">**最低层次**</mark>访问。使用Discovery Client，可以查询所有服务以及这些服务对应的URL。

## 1. 使用**@EnableDiscoveryClient**注解标注引导类

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

<mark style="color:blue;">**@EnableDiscoveryClient**</mark>注解是Spring Cloud的触发器，其作用是使应用程序能够使用Discovery Client和Spring Cloud LoadBalancer库。

## 2. 注入DiscoveryClient Bean对象

```java
private DiscoveryClient discoveryClient;

@Autowired
public void setDiscoveryClient(DiscoveryClient discoveryClient) {
    this.discoveryClient = discoveryClient;
}
```

DiscoveryClient的全限定名称是：<mark style="color:blue;">**org.springframework.cloud.client.discovery.DiscoveryClient**</mark>

## 3. 通过DiscoveryClient查找指定服务

{% code overflow="wrap" %}
```java
List<ServiceInstance> organizationInstances = discoveryClient
                .getInstances("organization-service");
```
{% endcode %}

## 4. 使用RestTemplate访问服务

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

> 在上述代码中实例化了RestTemplate类。
>
> 一旦在应用程序类中通过<mark style="color:blue;">**@EnableDiscoveryClient**</mark>启用了Spring Discovery Client，由Spring框架管理的所有Rest模板都将**向这些RestTemplate实例注入一个启用了Load Balancer的拦截器**。这个拦截器将改变使用RestTemplate类创建URL的行为。直接实例化RestTemplate类可以避免这种行为。
