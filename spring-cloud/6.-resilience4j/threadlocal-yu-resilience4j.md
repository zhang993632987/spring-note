# ThreadLocal与Resilience4j

We will define some values in ThreadLocal to see if they are propagated throughout the methods using Resilience4J annotations. Remember, Java ThreadLocal allows us to create variables that can be read and written to only by the same threads.

Let’s look at a concrete example. Often in a REST-based environment, we want to pass contextual information to a service call that will help us operationally manage the service. For example, we might pass a correlation ID or authentication token in the HTTP header of the REST call that can then be propagated to any downstream service calls. The correlation ID allows us to have a unique identifier that can be traced across multiple service calls in a single transaction.

To make this value available anywhere within our service call, we might use a Spring Filter class to intercept every call in our REST service. It can then retrieve this information from the incoming HTTP request and store this contextual information in a custom UserContext object. Then, anytime our code needs to access this value in our REST service call, our code can retrieve the UserContext from the ThreadLocal storage variable and read the value. Listing 7.12 shows an example of a Spring filter that we can use in our licensing service.

<details>

<summary>UserContextFilter</summary>

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

The UserContextHolder class stores the UserContext in a ThreadLocal class. Once it’s stored in ThreadLocal, any code that’s executed for a request will use the UserContext object stored in the UserContextHolder.

The following listing shows the UserContextHolder class.

<details>

<summary>UserContextHolder </summary>

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

The UserContext is a POJO class that contains all the specific data we want to store in the UserContextHolder. The following listing shows the content of this class.

<details>

<summary>UserContext </summary>

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

The last step that we need to do to finish our example is to add the logging instruction to the LicenseController class and LicenseServiceImpl class.：

<details>

<summary>LicenseController</summary>

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

<summary>LicenseServiceImpl </summary>

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

To execute our example, we’ll call our service, passing in a correlation ID using an HTTP header tmx-correlation-id and a value of TEST-CORRELATION-ID.

Once this call is submitted, we should see three log messages in the console, writing out the passed-in correlation ID as it flows through the UserContext, LicenseController, and LicenseService classes:

{% code overflow="wrap" %}
```log
UserContextFilter Correlation id: TEST-CORRELATION-ID
LicenseServiceController Correlation id: TEST-CORRELATION-ID
CircuitBreakerFactory Correlation id: 
```
{% endcode %}

```log
UserContextFilter Correlation id: TEST-CORRELATION-ID
LicenseServiceController Correlation id: TEST-CORRELATION-ID
Annotation Correlation id: TEST-CORRELATION-ID
```

If you don’t see the log messages on your console, add the code lines shown in the following listing to the application.yml or application.properties file of the licensingservice.

```yaml
logging:
  level:
    com.study: DEBUG
```

once the call hits the resiliency protected method, we still get the values written out for the correlation ID. This means that the parent thread values are available on the methods using the Resilience4j annotations.
