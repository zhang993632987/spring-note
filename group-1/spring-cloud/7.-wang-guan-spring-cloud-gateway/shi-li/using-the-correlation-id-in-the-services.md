# Using the correlation ID in the services

Now that we’ve guaranteed that a correlation ID has been added to every microservice call flowing through the gateway, we want to ensure that:

The correlation ID is readily accessible to the microservice that’s invoked.

Any downstream service calls the microservice might make also propagate the correlation ID on to the downstream calls.

To implement this, we’ll build a set of three classes for each of our microservices:UserContextFilter, UserContext, and UserContextInterceptor. These classes will work together to read the correlation ID of the incoming HTTP request, map it to a class that’s easily accessible and useable by the business logic in the application, and then ensure that the correlation ID is propagated to any downstream service calls. Figure 8.12 demonstrates how we will build these different pieces for our licensing service.

<figure><img src="../../../../.gitbook/assets/image (2).png" alt="" width="375"><figcaption></figcaption></figure>

When a call is made to the licensing service through the gateway, the TrackingFilter injects a correlation ID into the incoming HTTP header for any calls coming into the gateway.

The UserContextFilter class, a custom HTTP ServletFilter, maps a correlation ID to the UserContext class. The UserContext class stores the values in a thread for use later in the call.

The licensing service business logic executes a call to the organization service.

A RestTemplate invokes the organization service. The RestTemplate uses a custom Spring interceptor class, UserContextInterceptor, to inject the correlation ID into the outbound call as an HTTP header.

> Repeated code vs. shared libraries
>
> The subject of whether to use common libraries across your microservices is a gray area in microservice design. Microservice purists will tell you that you shouldn’t use a custom framework across your services because it introduces artificial dependencies. Changes in business logic or a bug can introduce wide-scale refactoring of all your services. On the other hand, other microservice practitioners will say that a purist approach is impractical because certain situations exist (like the previous UserContextFilter example) where it makes sense to build a common library and share it across services.
>
> We think there’s a middle ground here. Common libraries are fine when dealing with infrastructure-style tasks. If you start sharing business-oriented classes, you’re asking for trouble because you’ll end up breaking down the boundaries between the servicesUserContextFilter class is an HTTP servlet filter that will intercept all incoming HTTP requests coming into the service and map the correlation ID (and a few other values) from the HTTP request to the UserContext class.

在客户端弹性一章中，已经给出了 UserContextFilter、UserContext 和 UserContextHolder 的代码：

{% content-ref url="../../6.-ke-hu-duan-dan-xing-mo-shi-resilience4j/6.4-threadlocal-yu-resilience4j.md" %}
[6.4-threadlocal-yu-resilience4j.md](../../6.-ke-hu-duan-dan-xing-mo-shi-resilience4j/6.4-threadlocal-yu-resilience4j.md)
{% endcontent-ref %}

Custom RestTemplate and UserContextInterceptor: Ensuring that the correlation ID gets propagated

UserContextInterceptor class injects the correlation ID into any outgoing HTTP-based service request that’s executed from a RestTemplate instance. This is done to ensure that we can establish a link between service calls. To do this, we’ll use a Spring interceptor that’s injected into the RestTemplate class. Let’s look at the UserContextInterceptor in the following listing.

<details>

<summary></summary>



</details>

To use UserContextInterceptor, we need to define a RestTemplate bean and then add UserContextInterceptor to it. To do this, we’ll define our own RestTemplate bean in the LicenseServiceApplication class.

```java
@LoadBalanced    
@Bean
public RestTemplate getRestTemplate(){
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
```

With this bean definition in place, any time we use the @Autowired annotation and inject a RestTemplate into a class, we’ll use the RestTemplate created in listing 8.16 with the UserContextInterceptor attached to it.

> Log aggregation, authentication, and more
>
> Now that we have correlation IDs passed to each service, it’s possible to trace a transaction as it flows through all the services involved in the call. To do this, you need to ensure that each service logs to a central log aggregation point that captures log entries from all of your services into a single point. Each log entry captured in the log aggregation service will have a correlation ID associated with it.
