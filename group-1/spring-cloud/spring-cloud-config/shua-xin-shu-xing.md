# 刷新属性

Config服务器端始终提供最新版本的属性，通过其底层存储库，对属性进行的更改将是最新的。但是，<mark style="color:orange;">**Spring Boot应用程序只会在启动时读取它们的属性，因此Config服务器端中进行的属性更改不会被Spring Boot应用程序自动获取。**</mark>

Spring Boot Actuator提供了一个<mark style="color:blue;">**@RefreshScope**</mark>注解，允许开发团队访问<mark style="color:blue;">**/refresh**</mark>端点（POST），这会**强制Spring Boot应用程序重新读取其应用程序配置**。

```java
package com.study.cloudlearning;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.context.config.annotation.RefreshScope;

@RefreshScope
@SpringBootApplication
public class CloudLearningApplication {

    public static void main(String[] args) {
        SpringApplication.run(CloudLearningApplication.class, args);
    }
}
```

{% hint style="warning" %}
## <mark style="color:orange;">注意</mark>

**注解只会重新加载应用程序配置中的自定义Spring属性。**
{% endhint %}

> <mark style="color:blue;">**关于刷新微服务**</mark>
>
> <mark style="color:orange;">**将微服务与Spring Cloud Config服务一起使用时，在动态更改属性之前需要考虑的一件事是，可能会有同一服务的多个实例正在运行**</mark>。需要使用新的应用程序配置刷新所有这些服务。有几种方法可以解决这个问题。
>
> * Spring Cloud Config服务提供了一种称为<mark style="color:blue;">**Spring Cloud Bus**</mark>的基于推送的机制，使Spring Cloud Config服务器端能够向所有使用服务的客户端发布哪里的配置发生了更改的消息。Spring Cloud Bus需要一个额外的在运行的中间件：RabbitMQ。这是检测更改的非常有用的手段，但并不是所有的Spring Cloud Config后端都支持这种推送机制（例如，Consul服务器）。
> * 刷新Spring Cloud Config中的应用程序属性，然后<mark style="color:blue;">**编写一个简单的脚本来查询服务发现引擎以查找服务的所有实例，并直接调用/refresh端点**</mark>。
> * <mark style="color:blue;">**重新启动所有服务器或容器来接收新的属性**</mark>。这项工作很简单，特别是在Docker等容器服务中运行服务时。重新启动Docker容器差不多需要几秒，然后将强制重新读取应用程序配置。
>
> 记住，<mark style="color:red;">**基于云的服务器是短暂的。不要害怕使用新配置启动服务的新实例，将流量导向新的服务并拆除旧的服务。**</mark>
