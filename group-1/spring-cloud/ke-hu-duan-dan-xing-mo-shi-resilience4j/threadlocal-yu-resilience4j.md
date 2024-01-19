# ThreadLocal与Resilience4j

我们将在 ThreadLocal 中定义一些值，以查看它们是否能通过 Resilience4J 注解在方法之间传播。请记住，Java 的 ThreadLocal 允许我们创建只能由同一线程读取和写入的变量。

## 示例

让我们看一个具体的例子。在基于 REST 的环境中，我们经常希望将上下文信息传递给服务调用，以帮助我们在操作上管理服务。例如，我们可以在 REST 调用的 HTTP 头中传递关联 ID 或身份验证令牌，然后可以传播到任何下游服务调用。关联 ID 允许我们拥有一个在单个事务中跨多个服务调用进行跟踪的唯一标识符。

为了在我们的服务调用中的任何地方使此值可用，我们可能会使用 Spring 过滤器类来拦截我们 REST 服务中的每个调用。然后，它可以从传入的 HTTP 请求中检索此信息，并将此上下文信息存储在自定义的 UserContext 对象中。然后，每当我们的代码需要在我们的 REST 服务调用中访问此值时，可以从 ThreadLocal 存储变量中检索 UserContext 并读取该值。

UserContextFilter 类是一个在 licensing 服务中使用的 Spring 过滤器：

<details>

<summary><mark style="color:purple;">UserContextFilter</mark></summary>

```java
@Log4j2
@Component
public class UserContextFilter implements Filter {

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)
            throws IOException, ServletException {

        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;


        String correlationId = httpServletRequest.getHeader(UserContext.CORRELATION_ID);
        String userId = httpServletRequest.getHeader(UserContext.USER_ID);
        String authToken = httpServletRequest.getHeader(UserContext.AUTH_TOKEN);
        String organizationId = httpServletRequest.getHeader(UserContext.ORGANIZATION_ID);
        UserContext userContext = new UserContext(correlationId, authToken, userId, organizationId);
        UserContextHolder.setContext(userContext);

        log.debug("UserContextFilter Correlation id: {}", UserContextHolder.getContext().getCorrelationId());

        filterChain.doFilter(httpServletRequest, servletResponse);
    }
}
```

</details>

UserContextHolder 类将 UserContext 存储在 ThreadLocal 类中。一旦将一个 UserContext 对象存储在了  ThreadLocal 中，任何请求都可以使用 UserContextHolder 获取这个 UserContext 对象：

<details>

<summary><mark style="color:purple;">UserContextHolder</mark> </summary>

{% code overflow="wrap" %}
```java
public class UserContextHolder {
    private static final ThreadLocal<UserContext> userContext = new ThreadLocal<UserContext>();

    public static final UserContext getContext(){
        UserContext context = userContext.get();

        if (context == null) {
            context = createEmptyContext();
            userContext.set(context);

        }
        return userContext.get();
    }

    public static final void setContext(UserContext context) {
        Assert.notNull(context, "Only non-null UserContext instances are permitted");
        userContext.set(context);
    }

    public static final UserContext createEmptyContext(){
        return new UserContext();
    }
}
```
{% endcode %}

</details>

UserContext 是一个 POJO 类，包含我们想要存储在 UserContextHolder 中的所有数据，如下所示：

<details>

<summary><mark style="color:purple;">UserContext</mark> </summary>

```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Component
public class UserContext {
    public static final String CORRELATION_ID = "tmx-correlation-id";
    public static final String AUTH_TOKEN = "tmx-auth-token";
    public static final String USER_ID = "tmx-user-id";
    public static final String ORGANIZATION_ID = "tmx-organization-id";

    private String correlationId = "";
    private String authToken = "";
    private String userId = "";
    private String organizationId = "";
}
```

</details>

整个示例的最后一步是将日志记录指令添加到 LicenseController 类和 LicenseServiceImpl 类中：

<details>

<summary><mark style="color:purple;">LicenseController</mark></summary>

```java
@GetMapping
public Organization getOrganization(
        @PathVariable("organizationId") String organizationId,
        String type) {
    log.debug("LicenseServiceController Correlation id: {}",
            UserContextHolder.getContext().getCorrelationId());
    if ("annotation".equals(type)) {
        return licenseService.getOrganizationByAnnotation(organizationId);
    }
    return licenseService.getOrganizationByCBFactory(organizationId);
}
```

</details>

<details>

<summary><mark style="color:purple;">LicenseServiceImpl</mark> </summary>

```java
@Override
public Organization getOrganizationByCBFactory(String organizationId) {

    long start = System.currentTimeMillis();
    try {
        return circuitBreakerFactory.create(ORGANIZATION_SERVICE).run(
                () -> {
                    log.debug("CircuitBreakerFactory Correlation id: {}",
                            UserContextHolder.getContext().getCorrelationId());
                    return organizationFeignClient.getOrganization(organizationId);
                },
                throwable -> getOrgBackup(organizationId, throwable)
        );
    } catch (Exception e) {
        throw e;
    } finally {
        long end = System.currentTimeMillis();
        log.error("共花费：" + (end - start));
    }
}

@Override
@RateLimiter(name = "licenseService", fallbackMethod = "getOrgBackup")
@Retry(name = "retryLicenseService", fallbackMethod = "getOrgBackup")
@Bulkhead(name = ORGANIZATION_SERVICE, fallbackMethod = "getOrgBackup")
@CircuitBreaker(name = ORGANIZATION_SERVICE, fallbackMethod = "getOrgBackup")
public Organization getOrganizationByAnnotation(String organizationId) {
    log.debug("Annotation Correlation id: {}",
            UserContextHolder.getContext().getCorrelationId());
    return organizationFeignClient.getOrganization(organizationId);
}
```

</details>

## 结果

我们将调用我们的服务，并通过 HTTP 头 tmx-correlation-id 传递一个关联 ID，其值为 TEST-CORRELATION-ID。

一旦提交了这个调用，我们应该在控制台中看到三条日志消息，写出传递的关联 ID，它将通过 **UserContext**、**LicenseController** 和 **LicenseServiceImpl** 类流动：

<details>

<summary><mark style="color:purple;">Resilience4j 注解</mark></summary>

```properties
UserContextFilter Correlation id: TEST-CORRELATION-ID
LicenseServiceController Correlation id: TEST-CORRELATION-ID
Annotation Correlation id: TEST-CORRELATION-ID
```

</details>

<details>

<summary><mark style="color:purple;">CircuitBreakerFactory</mark></summary>

```properties
UserContextFilter Correlation id: TEST-CORRELATION-ID
LicenseServiceController Correlation id: TEST-CORRELATION-ID
CircuitBreakerFactory Correlation id: 
```

</details>

> 如果在控制台上看不到日志消息，请将以下配置添加到 licensing 服务的 application.yml 或 application.properties 文件中：
>
> ```yaml
> logging:
>   level:
>     com.study: DEBUG
> ```

## 结论

**“**<mark style="color:purple;">**Resilience4j 注解”**</mark>与**“**<mark style="color:purple;">**CircuitBreakerFactory”**</mark>这两种为代码添加客户端弹性保护的使用方式在 TeardLocal 值上表现出了不一样的行为：

* 对于利用**“**<mark style="color:purple;">**Resilience4j 注解”**</mark>标注的弹性保护方法，**ThreadLocal 中的关联 ID 依然能够保存下来**
* 但是当使用**“**<mark style="color:purple;">**CircuitBreakerFactory”**</mark>来为方法增加弹性保护时，**ThreadLocal 中的关联 ID 却**<mark style="color:orange;">**并未**</mark>**能能够保存下来**

> ## <mark style="color:orange;">CircuitBreakerFactory 添加UserContext</mark>&#x20;
>
> * 首先，**在当前线程中获取 UserContext，并将 UserContext 放入一个子线程能够获取的共享变量中**
> * 然后，**通过共享变量的方式将 UserContext 放入子线程的 ThreadLocal 中**
>
> 在此处使用的内部类与局部变量，如果不使用局部类，也可以使用成员变量或静态变量进行变量共享。
>
> ```java
> @Override
> public Organization getOrganizationByCBFactory(String organizationId) {
>     long start = System.currentTimeMillis();
>     final UserContext context = UserContextHolder.getContext();
>     try {
>         return circuitBreakerFactory.create(ORGANIZATION_SERVICE).run(
>                 () -> {
>                     UserContextHolder.setContext(context);
>                     log.debug("CircuitBreakerFactory Correlation id: {}",
>                             UserContextHolder.getContext().getCorrelationId());
>                     return organizationFeignClient.getOrganization(organizationId);
>                 },
>                 throwable -> getOrgBackup(organizationId, throwable)
>         );
>     } catch (Exception e) {
>         throw e;
>     } finally {
>         long end = System.currentTimeMillis();
>         log.error("共花费：" + (end - start));
>     }
> }
> ```
