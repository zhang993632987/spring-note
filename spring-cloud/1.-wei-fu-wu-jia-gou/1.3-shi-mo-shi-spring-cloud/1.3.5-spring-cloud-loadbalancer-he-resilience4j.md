# 1.3.5 Spring Cloud LoadBalancer和Resilience4j

对于微服务客户端弹性模式，Spring Cloud封装了<mark style="color:blue;">**Resilience4j**</mark>库和<mark style="color:blue;">**Spring Cloud LoadBalancer**</mark>项目。

* 通过使用Resilience4j库，你可以快速实现服务**客户端弹性模式**，如**断路器**、**重试**和**舱壁**等模式。
* Spring Cloud LoadBalancer项目简化了与诸如Eureka这样的服务发现代理的集成，同时它也为服务消费者提供了客户端对服务调用的**负载均衡**。这使得即使服务发现代理暂时不可用，客户端也可以继续进行服务调用。
