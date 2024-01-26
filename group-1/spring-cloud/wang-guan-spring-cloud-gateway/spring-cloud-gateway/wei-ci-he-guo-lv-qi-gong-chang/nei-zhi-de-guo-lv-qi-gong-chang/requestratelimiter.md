# RequestRateLimiter

**RequestRateLimiter 网关过滤器工厂使用一个 RateLimiter 实现来确定当前请求是否被允许继续进行。**<mark style="color:blue;">**如果不允许，将返回默认情况下的 HTTP 429 - Too Many Requests 状态。**</mark>

> <mark style="color:blue;">**此过滤器接受一个可选的 keyResolver 参数和特定于速率限制器的参数。**</mark>

## KeyResolver&#x20;

keyResolver 是一个实现 KeyResolver 接口的 bean。在配置中，可以通过 SpEL（Spring Expression Language）引用此 bean。例如：**#{@myKeyResolver}** 是一个 SpEL 表达式，引用了一个名为 myKeyResolver 的 bean。

以下代码展示了 KeyResolver 接口：

```java
public interface KeyResolver {

    /**
     * Resolve the key for the rate limiter.
     *
     * @param exchange the current exchange
     * @return the key for the rate limiter
     */
    Mono<String> resolve(ServerWebExchange exchange);
}
```

> <mark style="color:blue;">**KeyResolver 的默认实现是 PrincipalNameKeyResolver，它从 ServerWebExchange 中获取 Principal 并调用 Principal.getName()。**</mark>&#x20;

> <mark style="color:orange;">**默认情况下，如果 KeyResolver 找不到键，请求将被拒绝。**</mark>
>
> 可以通过设置 **spring.cloud.gateway.filter.request-rate-limiter.deny-empty-key**（true 或 false）和 **spring.cloud.gateway.filter.request-rate-limiter.empty-key-status-code** 属性来调整此行为。

## Redis RateLimiter

Redis 实现基于 [Stripe](https://stripe.com/blog/rate-limiters) 的工作，它要求使用 spring-boot-starter-data-redis-reactive。&#x20;

**Redis RateLimiter 使用的算法是**[**令牌桶算法**](https://en.wikipedia.org/wiki/Token\_bucket)**：**

* <mark style="color:blue;">**redis-rate-limiter.replenishRate**</mark> 属性定义了**每秒允许多少请求**（不包括任何被丢弃的请求）。这是**令牌桶填充的速率**。&#x20;
*   <mark style="color:blue;">**redis-rate-limiter.burstCapacity**</mark> 属性是**用户在一秒内允许的最大请求数**（不包括任何被丢弃的请求）。这是**令牌桶可以容纳的令牌数量**。

    > **将此值设置为零会阻止所有请求。**&#x20;
* <mark style="color:blue;">**redis-rate-limiter.requestedTokens**</mark> 属性是**每个请求消耗的令牌数**。这是每个请求从令牌桶中取走的令牌数，**默认为 1。**

> * 通过将 replenishRate 和 burstCapacity 设置为同样的值可以实现稳定的速率。&#x20;
> * 通过将 burstCapacity 设置得高于 replenishRate，可以允许临时的请求突发。在这种情况下，速率限制器需要在两个连续的突发之间留出一些时间（根据 replenishRate），因为连续的两个突发会导致请求被丢弃（HTTP 429 - Too Many Requests）。

> 将速率限制设置为低于 1 请求/秒的方式是将 replenishRate 设置为所需请求数，将 requestedTokens 设置为以秒为单位的时间跨度，将 burstCapacity 设置为 replenishRate 和 requestedTokens 的乘积。&#x20;
>
> 例如，设置 replenishRate=1，requestedTokens=60，和 burstCapacity=60 将得到一个 1 请求/分钟的限制。

以下配置示例演示了如何配置 redis-rate-limiter：

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: https://example.org
        filters:
        - name: RequestRateLimiter
          args:
            redis-rate-limiter.replenishRate: 10
            redis-rate-limiter.burstCapacity: 20
            redis-rate-limiter.requestedTokens: 1
```

上述配置定义了每个用户的请求速率限制为 10。允许突发量为 20，但在下一秒中只有 10 个请求可用。

以下示例演示了如何在 Java 中配置一个 KeyResolver：

```java
@Bean
KeyResolver userKeyResolver() {
    return exchange -> Mono.just(
        exchange.getRequest().getQueryParams().getFirst("user")
    );
}
```

KeyResolver 是一个简单的实现，它获取用户请求参数。&#x20;

还可以将速率限制器定义为实现 RateLimiter 接口的 bean。在配置中，可以使用 SpEL 引用该 bean。#{@myRateLimiter} 是一个 SpEL 表达式，引用了名为 myRateLimiter 的 bean。以下示例定义了一个速率限制器，它使用了前面示例中定义的 KeyResolver：

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: https://example.org
        filters:
        - name: RequestRateLimiter
          args:
            rate-limiter: "#{@myRateLimiter}"
            key-resolver: "#{@userKeyResolver}"
```
