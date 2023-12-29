# 使用服务发现进行自动路由映射

## 配置

Spring Cloud Gateway可以根据服务ID自动路由请求，只需将以下配置添加到gateway-server的配置文件中，如下所示：

```yaml
spring:
  cloud:
    gateway:
        # Enables the gateway to create routes based on services 
        # registered with service discovery
        discovery.locator:    
          enabled: true
          lowerCaseServiceId: true
```

通过添加以上配置，Spring Cloud Gateway将自动使用被调用服务的Eureka服务ID，并将其映射到下游服务实例。例如，如果我们想要调用我们的 organization 服务并通过Spring Cloud Gateway进行自动路由，我们将让客户端使用以下URL调用网关服务实例端点：

```properties
http://localhost:8072/organization-service/v1/organization/100
```

在这里，网关服务器通过 **http://localhost:8072** 端点访问。我们想要调用的服务（organization 服务）由端点路径中的第一部分表示。下图展示了这个映射的实际情况。

<figure><img src="../../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

使用Spring Cloud Gateway与Eureka的美妙之处在于，我们不仅可以通过单一端点进行调用，而且还可以添加和删除服务的实例，而无需修改网关。

## Actuator 查询

如果我们想查看由网关服务器管理的路由，可以通过在网关服务器上使用**actuator/gateway/routes**端点列出路由。这将返回我们服务上所有映射的列表：

<details>

<summary><mark style="color:purple;">http://localhost:8072/actuator/gateway/routes</mark></summary>

{% code overflow="wrap" %}
```json
[
    {
        "predicate": "Paths: [/license-service/**], match trailing slash: true",
        "metadata": {
            "management.port": "8080"
        },
        "route_id": "ReactiveCompositeDiscoveryClient_LICENSE-SERVICE",
        "filters": [
            "[[RewritePath /license-service/?(?<remaining>.*) = '/${remaining}'], order = 1]"
        ],
        "uri": "lb://LICENSE-SERVICE",
        "order": 0
    },
    {
        "predicate": "Paths: [/gateway-server/**], match trailing slash: true",
        "metadata": {
            "management.port": "8072"
        },
        "route_id": "ReactiveCompositeDiscoveryClient_GATEWAY-SERVER",
        "filters": [
            "[[RewritePath /gateway-server/?(?<remaining>.*) = '/${remaining}'], order = 1]"
        ],
        "uri": "lb://GATEWAY-SERVER",
        "order": 0
    },
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
    }
]
```
{% endcode %}

</details>
