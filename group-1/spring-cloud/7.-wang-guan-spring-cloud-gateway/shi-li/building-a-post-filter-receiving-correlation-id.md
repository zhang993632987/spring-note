# Building a post-filter receiving correlation ID

The following listing shows the code for building a post-filter.

```java
@Configuration
public class ResponseFilter {
   final Logger logger =LoggerFactory.getLogger(ResponseFilter.class);
   @Autowired
   FilterUtils filterUtils;
   @Bean
   public GlobalFilter postGlobalFilter() {
      return (exchange, chain) -> {
         return chain.filter(exchange).then(Mono.fromRunnable(() -> {
           HttpHeaders requestHeaders = exchange.getRequest().getHeaders();
           //Grabs the correlation ID that was passed in to the original HTTP request
           String correlationId = 
                filterUtils.
                getCorrelationId(requestHeaders);   
           logger.debug(
               "Adding the correlation id to the outbound headers. {}",
                         correlationId);
             // Injects the correlation ID into the response
               exchange.getResponse().getHeaders().
               add(FilterUtils.CORRELATION_ID,    
               correlationId);
           logger.debug("Completing outgoing request 
                          for {}.",   
                          exchange.getRequest().getURI());
           }));
         };
    }
}
```

Once we’ve implemented the ResponseFilter, we can fire up our service and call the licensing or organization service with it. Once the service completes, you’ll see a tmx-correlation-id on the HTTP response header from the call.

<details>

<summary></summary>



</details>
