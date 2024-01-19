# 重试模式

**重试模式负责在与服务的通信失败时进行重试尝试。**

* 该模式背后的关键概念是，通过尝试调用相同的服务一次或多次，来提供一种获取预期响应的方式。
* 对于此模式，我们必须指定给定服务实例的重试次数以及希望在每次重试之间经过的时间间隔。

## 配置

与断路器一样，Resilience4j允许我们指定要重试和不重试的异常。

```yaml
resilience4j.retry:
  instances:
    retryLicenseService:
      maxRetryAttempts: 5 # The maximum number of retry attempts
      waitDuration: 10000 # The wait duration between the retry attempts
      retry-exceptions: # The list of exceptions you want to retry
        - java.util.concurrent.TimeoutException
```

* <mark style="color:blue;">**maxRetryAttempts**</mark>：
  * 服务的最大重试次数。
  * 默认值为3。
* <mark style="color:blue;">**waitDuration**</mark>：
  * 重试尝试之间的等待持续时间。
  * 默认值为500毫秒。
* <mark style="color:blue;">**retry-exceptions**</mark>：
  * 将被重试的错误类列表。
  * 默认值为空。
* <mark style="color:blue;">**intervalFunction**</mark>：
  * 设置一个函数，用于在失败后更新等待间隔。
* <mark style="color:blue;">**retryOnResultPredicate**</mark>：
  * 配置一个谓词，用于评估是否应重试结果。
  * 如果希望重试，此谓词应返回true。
* <mark style="color:blue;">**retryOnExceptionPredicate**</mark>：
  * 配置一个谓词，用于评估是否应重试异常。
  * 与前一个谓词相同，如果要重试，我们必须返回true。
* <mark style="color:blue;">**ignoreExceptions**</mark>：
  * 设置一个被忽略且不会被重试的错误类列表。
  * 默认值为空。

## 使用

```java
@Override
@Retry(name = "retryLicenseService", fallbackMethod = "getOrgBackup")
@Bulkhead(name= ORGANIZATION_SERVICE, fallbackMethod = "getOrgBackup")
@CircuitBreaker(name = ORGANIZATION_SERVICE, fallbackMethod = "getOrgBackup")
public Organization getOrganizationByAnnotation(String organizationId) {
    return organizationFeignClient.getOrganization(organizationId);
}
```

## Actuator 端点查看

<details>

<summary><mark style="color:purple;">http://localhost:8080/actuator/retries</mark></summary>

```json
{
    "retries": [
        "retryLicenseService"
    ]
}
```

</details>

<details>

<summary><mark style="color:purple;">http://localhost:8080/actuator/retryevents</mark></summary>

```json
{
    "retryEvents": []
}
```

</details>
