# pre-filter

TrackingFilter, that will inspect all incoming requests to the gateway and determine whether there’s an HTTP header called tmx-correlation-id present in the request. The tmx-correlation-id header will contain a unique GUID(Globally Universal ID) that can be used to track a user’s request across multiple microservices. If the tmx-correlation-id isn’t present on the HTTP header, our gateway TrackingFilter will generate and set the correlation ID.



To create a global filter in the Spring Cloud Gateway, we need to implement the GlobalFilter class and then override the filter() method. This method contains the business logic that the filter implements.

We’ve implemented a class called FilterUtils, which encapsulates common functionality used by all the filters.&#x20;

To test this call, we can call our organization or licensing service.&#x20;

```
// Some code
```

Once the call is submitted, we should see a log message in the console that writes out the passed-in correlation ID as it flows through the filter:

```
gatewayserver_1     | 2020-04-14 22:31:23.835 DEBUG 1 --- [or-http-epoll-3] 
c.o.gateway.filters.TrackingFilter       : tmx-correlation-id generated in 
tracking filter: 735d8a31-b4d1-4c13-816d-c31db20afb6a.
```

> If you don’t see the message on your console, just add the code lines shown in the following listing to the bootstrap.yml configuration file of the Gateway server. Then build again and execute your microservices.
>
> ```
> logging:
>   level:
>     com.netflix: WARN
>     org.springframework.web: WARN
>     com.optimagrowth: DEBUG
> ```
