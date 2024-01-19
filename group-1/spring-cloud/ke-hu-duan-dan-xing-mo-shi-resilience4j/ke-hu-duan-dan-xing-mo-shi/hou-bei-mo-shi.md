# 后备模式

<mark style="color:blue;">**后备模式提供了一种替代方法，当远程服务调用失败时，服务消费者不会生成异常，而是执行替代代码路径并尝试通过其他方式执行操作。**</mark>调用方不会收到指示问题的异常，而是可能会被通知其请求需要稍后重试。

要使用Resilience4j实现回退策略，需要执行两个步骤：

1. **添加后备方法属性：**这涉及向 @CircuitBreaker 注解或其他相关注解添加一个后备方法属性。该属性应包含一个方法的名称**，**该方法在Resilience4j因故障而中断远程调用后将调用。
2.  **定义后备方法：**后备方法必须位于与由 @CircuitBreaker 保护的原始方法相同的类中。

    要在Resilience4j中创建后备方法，需要创建一个与原始函数具有相同签名的方法，再加上一个额外的异常参数（表示目标异常）。通过相同的签名，可以将所有参数从原始方法传递到后备方法。

```java
@Override
@CircuitBreaker(name = BACKEND_A, fallbackMethod = "getOrganizationBackup")
public Organization getOrganizationByAnnotation(String organizationId) {
    return organizationFeignClient.getOrganization(organizationId);
}

private Organization getOrganizationBackup(
        String organizationId, Throwable throwable) {
    Organization organization = new Organization();
    organization.setId(organizationId);
    organization.setName(throwable.getMessage());
    return organization;
}
```

> **在确定是否要实施后备策略时，请记住以下几点：**
>
> 1. **后备提供了一种在资源超时或失败时采取的行动方案：** 如果你发现自己使用后备方法来捕获超时异常，然后除了记录错误之外什么都没做，你应该考虑在你的服务调用周围使用标准的 try...catch 块：捕获异常并将记录错误的逻辑放在 try...catch 块中。
> 2. **注意使用后备方法时采取的操作：**如果在后备代码块中调用另一个分布式服务，可能需要用 @CircuitBreaker 包装后备方法（保持代码的防御性）。
