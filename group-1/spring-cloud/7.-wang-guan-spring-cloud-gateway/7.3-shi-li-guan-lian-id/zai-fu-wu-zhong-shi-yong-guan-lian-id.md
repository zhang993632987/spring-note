# 在服务中使用关联ID

我们已经确保每个流经网关的微服务调用都添加了关联ID，我们还希望确保：

1. 关联ID对所调用的微服务是可访问的。
2. 下游服务调用其他微服务时，关联ID也能够传播到被下游服务调用的微服务中

## 流程

为了实现这一目标，我们将为每个微服务构建一组三个类：UserContextFilter、UserContext和UserContextInterceptor。这些类将协同工作，读取传入HTTP请求的关联ID，将其映射到一个在应用程序业务逻辑中易于访问和使用的类，然后确保关联ID传播到任何下游服务调用。

下图展示了我们将如何为许可服务构建这些不同组件：

<figure><img src="../../../../.gitbook/assets/image (2) (1).png" alt="" width="375"><figcaption></figcaption></figure>

1. 当通过网关调用 licensing 服务时，TrackingFilter 会在传入网关的任何调用的HTTP头中注入关联ID。
2. UserContextFilter类是一个自定义的HTTP Servlet过滤器，将关联ID映射到UserContext类。UserContext类会在线程中存储这些值，以供稍后在调用中使用。
3. licensing 服务业务逻辑执行对 organization 服务的调用。
4. RestTemplate调用 organization 服务。RestTemplate使用自定义的Spring拦截器类UserContextInterceptor，将关联ID作为HTTP头注入到出站调用中。

> ## <mark style="color:blue;">重复代码 vs. 共享库</mark>
>
> 重复代码与共享库之间的权衡问题在微服务设计中是一个模糊的领域。
>
> * **微服务纯粹主义者**认为不应该在服务之间使用自定义框架，因为这引入了人为的依赖关系。**业务逻辑的更改或错误可能导致对所有服务进行广泛的重构。**
> * 另一方面，其他微服务从业者认为纯粹主义的方法不切实际，因为**存在某些情况**（比如前面的UserContextFilter示例）在这些情况下，**构建一个共同的库并在服务之间共享是有意义的**。
>
> 我们认为有一个中间地带。
>
> * **在处理基础设施任务时，使用共同库是可以的。**
> * **但如果开始共享面向业务的类，那么你最终会面临问题，因为这会打破服务之间的边界。**

### UserContextFilter、UserContext 和 UserContextHolder

在客户端弹性一章中，已经给出了 UserContextFilter、UserContext 和 UserContextHolder 的代码：

{% content-ref url="../../6.-ke-hu-duan-dan-xing-mo-shi-resilience4j/6.4-threadlocal-yu-resilience4j.md" %}
[6.4-threadlocal-yu-resilience4j.md](../../6.-ke-hu-duan-dan-xing-mo-shi-resilience4j/6.4-threadlocal-yu-resilience4j.md)
{% endcontent-ref %}

### UserContextInterceptor

UserContextInterceptor 类将关联ID注入到从 RestTemplate 实例执行的任何出站HTTP服务请求中。这是为了确保我们可以建立服务调用之间的关联。

为此，我们将使用一个Spring拦截器，并将其注入到RestTemplate 类中：

<details>

<summary><mark style="color:purple;">UserContextInterceptor</mark> </summary>

```java
@Component
public class UserContextInterceptor implements
        ClientHttpRequestInterceptor, RequestInterceptor {
    @Override
    public ClientHttpResponse intercept(
            HttpRequest request, byte[] body, 
            ClientHttpRequestExecution execution)
            throws IOException {

        UserContext context = UserContextHolder.getContext();
        String correlationId = context.getCorrelationId();
        String authToken = context.getAuthToken();

        HttpHeaders headers = request.getHeaders();
        headers.add(UserContext.CORRELATION_ID, correlationId);
        headers.add(UserContext.AUTH_TOKEN, authToken);

        return execution.execute(request, body);
    }

    @Override
    public void apply(RequestTemplate template) {
        UserContext context = UserContextHolder.getContext();
        String correlationId = context.getCorrelationId();
        String authToken = context.getAuthToken();

        template.header(UserContext.CORRELATION_ID, correlationId)
                .header(UserContext.AUTH_TOKEN, authToken);
    }
}
```

</details>

因为 **UserContextInterceptor 实现了 ClientHttpRequestInterceptor, RequestInterceptor 这两个接口**，所以它可以通过两种方式进行使用：

*   ## <mark style="color:blue;">**RestTemplate**</mark>

    要使用 **UserContextInterceptor**，我们需要定义一个**RestTemplate** bean，并将 UserContextInterceptor添加到其中。

    <pre class="language-java"><code class="lang-java"><strong>@LoadBalanced    
    </strong>@Bean
    public RestTemplate restTemplate(){
       RestTemplate template = new RestTemplate();
       List interceptors = template.getInterceptors();
       // Adds UserContextInterceptor to the RestTemplate instance
       if (interceptors==null){    
           template.setInterceptors(Collections.singletonList(
                   new UserContextInterceptor()));
       }else{
          interceptors.add(new UserContextInterceptor());
          template.setInterceptors(interceptors);
       }
       return template;
    }
    </code></pre>

    有了这个bean定义，每当我们使用@Autowired注解并将一个RestTemplate注入到一个类中时，我们将使用创建的带有UserContextInterceptor的RestTemplate。
*   ## <mark style="color:blue;">OpenFeign</mark>

    因为 **UserContextInterceptor 实现了 **<mark style="color:blue;">**RequestInterceptor**</mark>** 接口，所以 UserContextInterceptor 这个Bean 会自动注入到** OpenFeign 客户端中。

    正常使用 Feign 进行远程方法调用既可以完成关联 ID 的传播。

> ## 日志聚合、身份验证等功能
>
> 现在我们已经在每个服务中传递了关联ID，可以追踪事务在所有调用涉及的服务中的流动。为实现这一点，**确保每个服务都记录到一个中央日志聚合点，将所有服务的日志条目集中到一个地方**。在日志聚合服务中捕获的每个日志条目都将有一个关联ID。
