# Manually mapping routes using service discovery

Spring Cloud Gateway allows our code to be more fine-grained by allowing us to explicitly define route mappings rather than relying solely on the automated routes created with the Eureka service ID. Suppose we want to simplify the route by shortening the organization name rather than having our organization service accessed in the gateway via the default route, /organization-service/v1/organization/{organization-id}.You can do this by manually defining the mapping for the route in the configuration file /configserver/src/main/resources/config/gateway-server.yml, which is located in the Spring Cloud Configuration Server repository. The following listing shows you how.

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

By adding this configuration, we can now access the organization service by entering the /organization/v1/organization/{organization-id} route. Now, if we recheck the Gateway server’s endpoint, we should see the results shown in figure 8.7.

If you look carefully at figure 8.7, you’ll notice that two entries are present for the organization service. One service entry is the mapping we defined in the gateway-server.yml file, which is organization/\*\***:** organization-service. The other service entry is the automatic mapping created by the gateway based on the Eureka ID for the organization service, which is /organization-service/\*、\*: organization-service.

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

> When we use automated route mapping where the gateway exposes the service based solely on the Eureka service ID, if no service instances are running, the gateway will not expose the route for the service. However, if we manually map a route to a service discovery ID and there are no instances registered with Eureka, the gateway will still show the route. If we try to call the route for the nonexistent service, it will return a 500 HTTP error.

If we want to exclude the automated mapping of the Eureka service ID route and only have the organization service route that we’ve defined, we can remove the spring.cloud.gateway.discovery.locator entries we added in the gateway-server.yml file.

Now, when we call the actuator/gateway/routes endpoint on the Gateway server, we should only see the organization service mapping we’ve defined. Figure 8.8 shows the outcome of this mapping.

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
