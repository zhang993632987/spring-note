# 6.2 客户端负载均衡模式

<mark style="color:blue;">**客户端负载均衡**</mark>涉及让客户端从**服务发现代理**（如Netflix Eureka）中查找服务的所有实例，然后<mark style="color:blue;">**缓存**</mark>服务实例的物理位置。

当服务消费者需要调用服务实例时，客户端负载均衡器将从它维护的服务位置池返回一个位置。因为客户端负载均衡器位于服务客户端和服务消费者之间，所以**负载均衡器可以检测服务实例是否抛出错误或表现不佳**。如果客户端负载均衡器**检测到问题**，它可以**从可用服务位置池中移除该服务实例**，并防止将来的服务调用访问该服务实例。

这正是Spring Cloud LoadBalancer库提供的**开箱即用**的功能（不需要额外的配置）。

> 如果直接使用 Load Balancer 内置的缓存方案，在服务启动时会看到以下警告信息：
>
> {% code overflow="wrap" %}
> ```log
> 2023-12-09 11:58:54.116  WARN 6136 --- [  restartedMain] iguration$LoadBalancerCaffeineWarnLogger : Spring Cloud LoadBalancer is currently working with the default cache. While this cache implementation is useful for development and tests, it's recommended to use Caffeine cache in production.You can switch to using Caffeine cache, by adding it and org.springframework.cache.caffeine.CaffeineCacheManager to the classpath.
> ```
> {% endcode %}
>
> 消除上述警告信息的办法时引入Caffeine作为Load Balancer的缓存方案：
>
> ```xml
> <dependency>
>     <groupId>com.github.ben-manes.caffeine</groupId>
>     <artifactId>caffeine</artifactId>
> </dependency>
> ```
>
> 只要引入上述依赖后，spring boot 会自动注入 CaffeineCacheManager Bean（通过 actuator 的 beans 端口查看）：
>
> ```json
> "caffeineLoadBalancerCacheManager": {
>     "aliases": [],
>     "scope": "singleton",
>     "type": "org.springframework.cloud.loadbalancer.cache.CaffeineBasedLoadBalancerCacheManager",
>     "resource": "class path resource [org/springframework/cloud/loadbalancer/config/LoadBalancerCacheAutoConfiguration$CaffeineLoadBalancerCacheManagerConfiguration.class]",
>     "dependencies": [
>         "org.springframework.cloud.loadbalancer.config.LoadBalancerCacheAutoConfiguration$CaffeineLoadBalancerCacheManagerConfiguration",
>         "spring.cloud.loadbalancer.cache-org.springframework.cloud.loadbalancer.cache.LoadBalancerCacheProperties"
>     ]
> },
> ```
