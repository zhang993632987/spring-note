# 使用服务发现进行手动路由映射

Spring Cloud Gateway通过允许我们显式定义路由映射，而不仅仅依赖于使用Eureka服务ID创建的自动化路由，使我们的代码更加灵活。

假设我们想通过简化组织名称来简化路由，而不是让组织服务通过默认路由（/organization-service/v1/organization/{organization-id}）在网关中访问。我们可以通过在配置文件 /configserver/src/main/resources/config/gateway-server.yml 中手动定义路由映射来实现这一点，该文件位于Spring Cloud配置服务器存储库中。以下是配置文件的示例：

```yaml
spring:
  cloud:
    gateway:
        discovery.locator:
          enabled: true
          lowerCaseServiceId: true
        routes:
        - id: organization-service # This optional ID is an arbitrary route ID.
          uri: lb://organization-service # Sets the route’s destination URI
          predicates:    
          - Path=/organization/**
          # Filters a collection of Spring web.filters to modify the 
          # request or response before or after sending the response
          filters: 
          # Rewrites the request path, from /organization/** to /**, 
          # by taking the path regexp as a parameter and a replacement order
          - RewritePath=/organization/(?<path>.*), /$\{path}
```

通过添加这个配置，我们现在可以通过输入 /organization/v1/organization/{organization-id} 路由来访问 organization 服务。现在，如果重新检查网关服务器的端点，我们应该看到如下的结果：

<details>

<summary><mark style="color:purple;">http://localhost:8072/actuator/gateway/routes</mark></summary>

{% code overflow="wrap" %}
```json
[
    ...
    {
        "predicate": "Paths: [/organization-service/**], match trailing slash: true",
        "metadata": {
            "management.port": "8081"
        },
        "route_id": "ReactiveCompositeDiscoveryClient_ORGANIZATION-SERVICE",
        "filters": [
            "[[RewritePath /organization-service/?(?<remaining>.*) = '/${remaining}'], order = 1]"
        ],
        "uri": "lb://ORGANIZATION-SERVICE",
        "order": 0
    },
    {
        "predicate": "Paths: [/organization/**], match trailing slash: true",
        "route_id": "organization-service",
        "filters": [
            "[[RewritePath /organization/(?<path>.*) = '/${path}'], order = 1]"
        ],
        "uri": "lb://organization-service",
        "order": 0
    }
]
```
{% endcode %}

</details>

仔细观察，您会注意到 organization 服务有两个条目：

* 一个服务条目是我们在 gateway-server.yml 文件中定义的映射，即 organization/\*\*: organization-service。
* 另一个服务条目是网关基于 organization 服务的Eureka ID创建的自动映射，即 /organization-service/\*\*: organization-service。

> 如果想排除Eureka服务ID路由的自动映射，只保留手动定义的组织服务路由，可以删除gateway-server.yml 文件中添加的 **spring.cloud.gateway.discovery.locator** 条目。

现在，当我们在Gateway服务器上调用 **actuator/gateway/routes** 端点时，应该只看到我们定义的组织服务映射：

<details>

<summary><mark style="color:purple;">http://localhost:8072/actuator/gateway/routes</mark></summary>

{% code overflow="wrap" %}
```json
[
    {
        "predicate": "Paths: [/organization/**], match trailing slash: true",
        "route_id": "organization-service",
        "filters": [
            "[[RewritePath /organization/(?<path>.*) = '/${path}'], order = 1]"
        ],
        "uri": "lb://organization-service",
        "order": 0
    }
]
```
{% endcode %}

</details>

{% hint style="info" %}
## <mark style="color:blue;">提示</mark>

* 当我们使用**自动路由映射**时，即网关仅根据Eureka服务ID公开服务时，**如果没有运行的服务实例，网关将不会暴露服务的路由**。
* 然而，**如果我们手动将路由映射到服务发现ID，而Eureka中没有注册任何实例，网关仍将显示该路由**。如果我们尝试调用不存在的服务路由，将返回一个500的HTTP错误。
{% endhint %}
