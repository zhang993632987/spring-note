# 预过滤器

我们将创建一个名为 **TrackingFilter** 的过滤器，它将检查所有传入网关的请求，并确定请求中是否存在一个名为 tmx-correlation-id 的HTTP头。tmx-correlation-id 头将包含一个唯一的全局标识符（GUID），可用于跟踪用户在多个微服务之间的请求。如果HTTP头中不存在tmx-correlation-id，TrackingFilter 将生成并设置关联ID。

要在Spring Cloud Gateway中创建全局过滤器，我们需要**实现 GlobalFilter 类，然后覆盖 filter() 方法**。这个方法包含了过滤器实现的业务逻辑。

```java
@Log4j2
@Order(1)
@Component
public class TrackingFilter implements GlobalFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String correlationID = FilterUtils.getCorrelationId(
                exchange.getRequest().getHeaders());
        if (correlationID == null) {
            correlationID = generateCorrelationId();
            exchange = FilterUtils.setCorrelationId(exchange, correlationID);
            log.debug("tmx-correlation-id generated in tracking filter: {}.",
                    correlationID);
        } else {
            log.debug("tmx-correlation-id found in tracking filter: {}. ",
                    correlationID);
        }
        return chain.filter(exchange);
    }

    private String generateCorrelationId() {
        return UUID.randomUUID().toString();
    }
}
```

我们已经实现了一个名为 **FilterUtils** 的类，该类封装了所有过滤器共用的常见功能。

```java
@Log4j2
public final class FilterUtils {

    public static final String CORRELATION_ID = "tmx-correlation-id";
    public static final String AUTH_TOKEN = "tmx-auth-token";
    public static final String USER_ID = "tmx-user-id";
    public static final String ORG_ID = "tmx-org-id";
    public static final String PRE_FILTER_TYPE = "pre";
    public static final String POST_FILTER_TYPE = "post";
    public static final String ROUTE_FILTER_TYPE = "route";

    public static String getCorrelationId(HttpHeaders headers) {
        List<String> correlationIds = headers.get(CORRELATION_ID);
        if (correlationIds != null) {
            return correlationIds.stream().findFirst().get();
        }
        return null;
    }

    public static ServerWebExchange setRequestHeader(
            ServerWebExchange exchange, String name, String value) {
        ServerHttpRequest request = exchange.getRequest().mutate()
                .header(name, value)
                .build();
        return exchange.mutate()
                .request(request)
                .build();
    }

    public static ServerWebExchange setCorrelationId(
            ServerWebExchange exchange, String correlationId) {
        return setRequestHeader(exchange, CORRELATION_ID, correlationId);
    }
}
```

为了测试这个调用，我们可以调用我们的 organization 或 licensing 服务：

```log
http://localhost:8072/organization-service/v1/organization/100
```

一旦调用被提交，我们应该在控制台中看到一个日志消息，该消息会在流经过滤器时输出传入的关联ID：

```
tmx-correlation-id generated in tracking filter: 3fef7f9d-0234-4a16-9736-22f5e964
```

> 如果在控制台上看不到消息，只需将以下清单中显示的代码行添加到网关服务器的 application.yml 配置文件中：
>
> ```yaml
> logging:
>   level:
>     com.netflix: WARN
>     org.springframework.web: WARN
>     com.study: DEBUG
> ```
