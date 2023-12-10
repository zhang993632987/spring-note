# 舱壁模式

## 介绍

如果不使用舱壁模式，默认行为是使用为整个Java容器处理请求而保留的相同线程去执行这些调用。在高负载情况下，多个服务中的一个性能问题可能导致用于Java容器的所有线程都被用满并等待处理工作，而新的工作请求则排队等待。**Java容器最终会崩溃**。

舱壁模式将远程资源调用隔离在它们自己的线程池中，以便单个不良行为的服务可以被隔离并且不会使容器崩溃。Resilience4j提供了舱壁模式的两种不同实现。

* <mark style="color:blue;">**信号量舱壁（Semaphore bulkhead）**</mark>**：** 使用信号量隔离方法，限制对服务的并发请求数量。一旦达到限制，它就开始拒绝请求。
* <mark style="color:blue;">**线程池舱壁（Thread pool bulkhead）**</mark>**：** 使用有界队列和固定线程池。这种方法只有在线程池和队列都满时才拒绝请求。

Resilience4j默认使用信号量舱壁。

## 配置

```yaml
resilience4j.bulkhead:
  instances:
    licenseService:
      maxWaitDuration: 10ms # The maximum amount of time to block a thread
      maxConcurrentCalls: 20 # The maximum number of concurrent calls

resilience4j.thread-pool-bulkhead:
  configs:
    shared:
      maxThreadPoolSize: 4 # The maximum number of threads in the thread pool
      coreThreadPoolSize: 2 # The core thread pool size
      queueCapacity: 2 # The queue’s capacity
      # The maximum time that idle threads wait for new tasks before terminating
      keepAliveDuration: 20ms 
  instances:
    organizationService:
      baseConfig: shared
```

* **maxWaitDuration（最大等待时间）**： 设置进入舱壁时阻塞线程的最长时间。默认值为0。
* **maxConcurrentCalls（最大并发调用数）**： 设置舱壁允许的最大并发调用数。默认值为25。
* **maxThreadPoolSize（最大线程池大小）**： 设置最大线程池大小。默认值为 Runtime.getRuntime().availableProcessors()。
* **coreThreadPoolSize（核心线程池大小）**： 设置核心线程池大小。默认值为 Runtime.getRuntime().availableProcessors()。
* **queueCapacity（队列容量）**： 设置队列的容量。默认值为100。
* **keepAliveDuration（保持存活时间）**： 设置空闲线程在终止之前等待新任务的最长时间。当线程数量高于核心线程数量时会发生。默认值为20毫秒。

> ## 一个自定义线程池的适当大小可以通过以下公式确定：
>
> {% code overflow="wrap" %}
> ```plsql
> (服务健康时的峰值每秒请求数 * 第99百分位延迟（以秒为单位）) + 一小部分额外线程以应对潜在的开销或负荷波动
> ```
> {% endcode %}
>
> 通常，在服务承受负载之前，我们往往不了解服务的性能特性。<mark style="color:blue;">**线程池属性可能需要调整的一个重要信号是：即使目标远程资源处于健康状态，服务调用也处于超时的过程中。**</mark>根据给定的公式调整线程池有助于确保线程池足够大，能够处理预期的负载并保持响应性，避免出现超时和服务性能下降等问题。

## 使用

### @bulkhead 注解

```java
@Override
@Bulkhead(name= ORGANIZATION_SERVICE, fallbackMethod = "getOrgBackup")
@CircuitBreaker(name = ORGANIZATION_SERVICE, fallbackMethod = "getOrgBackup")
public Organization getOrganizationByAnnotation(String organizationId) {
    return organizationFeignClient.getOrganization(organizationId);
}
```

### CircuitBreakerFactory

```java
Organization organization = circuitBreakerFactory.create(ORGANIZATION_SERVICE).run(
        () -> organizationFeignClient.getOrganization(organizationId),
        throwable -> getOrganizationBackup(organizationId, throwable)
);
```

> ## <mark style="color:orange;">注意：</mark>
>
> ### <mark style="color:blue;">circuitBreakerFactory 自动注入了与 circuitbreaker 同名的 bulkhead。</mark>

## Actuator 端点查看

<details>

<summary>http://localhost:8080/actuator/bulkheads</summary>

```json
{
    "bulkheads": [
        "licenseService",
        "organizationService"
    ]
}
```

</details>

<details>

<summary>http://localhost:8080/actuator/bulkheadevents</summary>

```json
{
    "bulkheadEvents": [
        {
            "bulkheadName": "organizationService",
            "type": "CALL_PERMITTED",
            "creationTime": "2023-12-10T15:33:44.013023100+08:00[Asia/Shanghai]"
        },
        {
            "bulkheadName": "organizationService",
            "type": "CALL_FINISHED",
            "creationTime": "2023-12-10T15:33:59.058528600+08:00[Asia/Shanghai]"
        }
    ]
}
```

</details>
