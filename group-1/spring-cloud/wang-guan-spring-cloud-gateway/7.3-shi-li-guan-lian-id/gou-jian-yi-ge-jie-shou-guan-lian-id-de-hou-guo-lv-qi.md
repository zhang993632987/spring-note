# 构建一个接收关联ID的后过滤器

以下清单展示了构建后过滤器（post-filter）的代码。

```java
@Log4j2
@Component
public class ResponseFilter implements GlobalFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        Runnable runnable = () -> postGlobalFilter(exchange);
        return chain.filter(exchange).then(Mono.fromRunnable(runnable));
    }

    private void postGlobalFilter(ServerWebExchange exchange) {
        HttpHeaders requestHeaders = exchange.getRequest().getHeaders();
        String correlationId = FilterUtils.getCorrelationId(requestHeaders);
        log.debug("Adding the correlation id to the outbound headers. {}",
                correlationId);
        exchange.getResponse().getHeaders().add(
                FilterUtils.CORRELATION_ID, correlationId);
        log.debug("Completing outgoing request for {}.",
                exchange.getRequest().getURI());
    }
}
```

一旦我们实现了ResponseFilter，我们就可以启动我们的服务并使用它调用许可或组织服务。一旦服务完成，您将在调用的HTTP响应头中看到tmx-correlation-id。

```log
http://localhost:8072/organization-service/v1/organization/100
```

<figure><img src="../../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```log
Adding the correlation id to the outbound headers. 3fef7f9d-0234-4a16-9736-22f5e9644f60
Completing outgoing request for http://localhost:8072/v1/organization/100.
```
{% endcode %}
