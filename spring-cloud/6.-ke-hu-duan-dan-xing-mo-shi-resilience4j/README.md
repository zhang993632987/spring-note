# 6. Resilience4j

<mark style="color:blue;">**Resilience4j**</mark>是一个受Hystrix启发的容错库。它提供了以下模式，以提高网络问题或多个服务故障的容错能力：

* <mark style="color:blue;">**断路器(CircuitBreaker)**</mark>——当被调用的服务发生失败时，停止发出请求。
* <mark style="color:blue;">**重试(Retry)**</mark>——在服务暂时失败时重试服务。
* <mark style="color:blue;">**舱壁(Bulkhead)**</mark>——限制传出的并发服务请求数以避免过载。
* <mark style="color:blue;">**限流(RateLimiter)**</mark>——限制一个服务在一定时间内接收的调用数。
* <mark style="color:blue;">**后备**</mark>——为失败的请求设置备用路径。

通过使用Resilience4j，我们可以通过定义<mark style="color:blue;">**方法的注解**</mark>，将几种模式应用到相同的方法调用中。如果我们想用舱壁模式和断路器模式限制传出调用的数量，我们可以为该方法定义**@CircuitBreaker**和**@Bulkhead**注解。重要的是要注意，Resilience4j的重试顺序如下：

```java
Retry ( CircuitBreaker ( RateLimiter ( TimeLimiter ( Bulkhead ( Function ) ) ) ) )
```

## 依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>
```
