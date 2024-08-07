# 使用带有 Load Balancer 功能的 RestTemplate 调用服务

要使用带有 Load Balancer 功能的 RestTemplate 类，需要使用 Spring Cloud 注解 <mark style="color:blue;">**@LoadBalanced**</mark> 来定义一个 RestTemplate Bean。

## 1. 定义 RestTemplate Bean

```java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

## 2. 使用上述 Bean

```java
private RestTemplate restTemplate;

private Organization getOrganization(String organizationId) {
        Organization organization = restTemplate.getForObject(
                "http://{applicationId}/v1/organization/{organizationId}",
                Organization.class,
                "organization-service",
                organizationId);
        return organization;
}

@Autowired
public void setRestTemplate(RestTemplate restTemplate) {
    this.restTemplate = restTemplate;
}
```

除了在定义目标服务的 URL 上有一点小小的差异，使用支持 Load Balancer 的 RestTemplate 类几乎和使用标准的 Spring RestTemplate 类一样。唯一的区别是需要<mark style="color:blue;">**使用要调用的服务的 Eureka 服务 ID 来构建目标 URL，而不是在 RestTemplate 调用中使用服务的物理位置。**</mark>

启用 Load Balancer 的 RestTemplate 类将解析传递给它的 URL，并使用传递的内容作为服务器名称，该服务器名称作为从 Load Balancer 查询服务实例的键。实际的服务位置和端口与开发人员完全抽象隔离。此外，**通过使用 RestTemplate 类，Spring Cloud LoadBalancer 将在所有服务实例之间使用**<mark style="color:blue;">**轮询算法**</mark>从而达到**请求的**<mark style="color:blue;">**负载均衡**</mark>**。**
