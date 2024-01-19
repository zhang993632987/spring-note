# 限流模式

## 介绍

Resilience4j提供了两种限流模式的实现：<mark style="color:blue;">**AtomicRateLimiter**</mark> 和 <mark style="color:blue;">**SemaphoreBasedRateLimiter**</mark>。

限流模式的默认实现是 **AtomicRateLimiter**。

### **SemaphoreBasedRateLimiter**

SemaphoreBasedRateLimiter 是最简单的实现。该实现使用一个 java.util.concurrent.Semaphore 存储当前的许可。

在这种情况下，所有用户线程将调用 semaphore.tryAcquire 方法来触发对额外的内部线程的调用，该线程在新的 limitRefreshPeriod 开始时执行 semaphore.release。

### **AtomicRateLimiter**&#x20;

与 SemaphoreBasedRateLimiter 不同，AtomicRateLimiter 不需要线程管理，因为用户线程本身执行所有许可逻辑。AtomicRateLimiter 将从启动开始的所有纳秒划分为周期，每个周期的持续时间是刷新周期（以纳秒为单位）。然后，在每个周期的开始，我们应该设置活动许可以限制该周期。

为了更好地理解这种方法，让我们看一下以下设置：

* **ActiveCycle**：最后一次调用使用的周期号
* **ActivePermissions**：最后一次调用后的可用许可数
* **NanosToWait**：最后一次调用等待许可的纳秒数

这个实现包含一些巧妙的逻辑：

* 周期是相等的时间片。
* 如果可用的许可不足，可以通过减少当前许可并计算需要等待许可出现的时间来执行许可预约。Resilience4j 允许通过定义在时间段内允许的调用次数（**limitForPeriod**）、许可刷新的频率（**limitRefreshPeriod**）以及线程等待获取许可的时间（**timeoutDuration**）来实现这一点。

## 配置

对于这种模式，必须指定<mark style="color:blue;">**超时持续时间**</mark>、<mark style="color:blue;">**限制刷新**</mark>**和**<mark style="color:blue;">**限制周期**</mark>。以下示例显示了包含重试配置参数的许可服务的 application.yml。

```yaml
resilience4j.ratelimiter:
  instances:
    licenseService:
      timeoutDuration: 1000ms # Defines the time a thread waits for permission
      limitRefreshPeriod: 5000 # Defines the period of a limit refresh
      # Defines the number of permissions available during a limit refresh period
      limitForPeriod: 5
```

* <mark style="color:blue;">**timeoutDuration**</mark>：
  * 线程等待许可的时间。
  * 默认值为5秒。
* <mark style="color:blue;">**limitRefreshPeriod**</mark>：
  * 限制刷新的周期。每个周期结束后，速率限制器将许可计数重置为 limitForPeriod 的值。
  * 默认值为500纳秒。
* <mark style="color:blue;">**limitForPeriod**</mark>：
  * 在一个刷新周期内可用的许可数。
  * 默认值为50。

## 使用

```java
@Override
@RateLimiter(name = "licenseService", fallbackMethod = "getOrgBackup")
@Retry(name = "retryLicenseService", fallbackMethod = "getOrgBackup")
@Bulkhead(name= ORGANIZATION_SERVICE, fallbackMethod = "getOrgBackup")
@CircuitBreaker(name = ORGANIZATION_SERVICE, fallbackMethod = "getOrgBackup")
public Organization getOrganizationByAnnotation(String organizationId) {
    return organizationFeignClient.getOrganization(organizationId);
}
```

## 舱壁模式和限流模式的区别

主要区别在于：

* <mark style="color:blue;">**舱壁模式负责限制并发调用的数量（例如，它每次只允许X个并发调用）。**</mark>
* <mark style="color:blue;">**速率限制器则可以限制在给定时间内的总调用次数（例如，每隔Y秒允许X次调用）。**</mark>

为了选择哪种模式适合您，仔细检查你的需求：

* 如果你想要阻塞并发调用，最好选择舱壁模式；
* 如果你想要限制在特定时间段内的总调用次数，最好选择速率限制器。
* 如果你同时考虑这两种情况，你也可以将它们结合使用。

## Actuator 端点查看

<details>

<summary><mark style="color:blue;">http://localhost:8080/actuator/ratelimiters</mark></summary>

```json
{
    "rateLimiters": [
        "licenseService"
    ]
}
```

</details>

<details>

<summary><mark style="color:blue;">http://localhost:8080/actuator/ratelimiterevents</mark></summary>



</details>
