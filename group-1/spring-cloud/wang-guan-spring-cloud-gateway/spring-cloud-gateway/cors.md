# CORS

## 全局 CORS 配置 <a href="#global-cors-configuration" id="global-cors-configuration"></a>

```yaml
spring:
  cloud:
    gateway:
      globalcors:
        add-to-simple-url-handler-mapping: true
        cors-configurations:
          '[/**]':
            allowedOrigins: "https://docs.spring.io"
            allowedMethods:
            - GET
```

在上述示例中，允许来自 docs.spring.io 发起的CORS请求，适用于所有 GET 请求的路径。&#x20;

**如果想要为那些未被某个网关路由谓词处理的请求提供相同的 CORS 配置，您可以将spring.cloud.gateway.globalcors.add-to-simple-url-handler-mapping 属性设置为true。**这个属性的作用是将全局的CORS配置添加到简单的URL处理映射中。这在处理 CORS 预检请求时非常有用，特别是当您的路由谓词不返回 true，因为 HTTP 方法是 OPTIONS 时。简而言之，<mark style="color:blue;">**这个属性的设置允许全局的 CORS 配置应用于未被特定网关路由处理的请求。**</mark>

## 路由 CORS 配置 <a href="#route-cors-configuration" id="route-cors-configuration"></a>

“路由”配置允许直接将 CORS 作为元数据应用于一个路由，使用键名为 cors。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: cors_route
        uri: https://example.org
        predicates:
        - Path=/service/**
        metadata:
          cors:
            allowedOrigins: '*'
            allowedMethods:
              - GET
              - POST
            allowedHeaders: '*'
            maxAge: 30
```

**如果路由中没有路径谓词（Path predicate），则将应用'/\*\*'作为默认路径。**
