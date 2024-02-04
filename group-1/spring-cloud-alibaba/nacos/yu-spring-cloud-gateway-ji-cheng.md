# 与 Spring Cloud Gateway 集成

与 Eureka 不同，<mark style="color:orange;">**当 Spring Cloud Gateway 使用 Nacos 作为服务发现时，无论是选择自动路由映射还是手动路由映射，都需要引入 spring-cloud-starter-loadbalancer 依赖**</mark>，如下所示：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

如果缺少上述依赖，当通过 gateway 访问服务时，会得到如下的异常信息：

```properties
There was an unexpected error (type=Service Unavailable, status=503).
```

{% hint style="warning" %}
## <mark style="color:orange;">**注意**</mark>

**与 OpenFeign 集成时，同样需要引入 spring-cloud-starter-loadbalancer 依赖，否则服务在启动时会报错。**
{% endhint %}
