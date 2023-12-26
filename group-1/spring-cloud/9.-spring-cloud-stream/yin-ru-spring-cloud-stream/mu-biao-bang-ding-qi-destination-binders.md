# 目标绑定器（Destination Binders）

Spring Cloud Stream 提供了 Binder 抽象，用于连接到外部中间件。

## Binder 检测 <a href="#_binder_detection" id="_binder_detection"></a>

Binder Detection（Binder 检测）是 Spring Cloud Stream 的一个特性，用于自动检测和选择合适的 Binder 实现，以连接到外部消息中间件。

### 类路径检测

默认情况下，Spring Cloud Stream 依赖于 Spring Boot 的自动配置机制，通过类路径上的相关库来检测可用的 Binder。如果在类路径上找到了唯一的 Binder 实现，Spring Cloud Stream 将自动使用它。

例如，一个旨在只绑定到 RabbitMQ 的 Spring Cloud Stream 项目可以添加以下依赖：

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
```

### 类路径上存在多个 Binder 实现 <a href="#multiple-binders" id="multiple-binders"></a>

当类路径上存在多个 Binder 实现时，应用程序必须指定要为每个目标绑定使用哪个 Binder：

* 可以通过 **spring.cloud.stream.defaultBinder** 属性在全局范围内进行 Binder 选择（例如，**spring.cloud.stream.defaultBinder=rabbit**）
*   也可以通过在每个绑定上配置 Binder 来进行选择。例如，一个处理器应用程序（具有分别用于读取和写入的命名为 **input** 和 **output** 的绑定）从 Kafka 读取并写入 RabbitMQ 可以指定以下配置：

    ```properties
    spring.cloud.stream.bindings.input.binder=kafka
    spring.cloud.stream.bindings.output.binder=rabbit
    ```

### 连接到多个系统 <a href="#multiple-systems" id="multiple-systems"></a>

默认情况下，binders 共享应用程序的 Spring Boot 自动配置，因此**在类路径上找到的每个 binder 实现都会创建一个实例**。如果**应用程序需要连接到同一类型的多个代理服务器，可以指定多个 binder 配置，并为每个配置提供不同的环境设置。**

> 打开显式的 binder 配置将完全禁用默认的 binder 配置过程。此时，使用的所有 binder 必须进行显示配置。
>
> 一些打算透明使用 Spring Cloud Stream 的框架可能会创建可以通过名称引用的 binder 配置，而且它们不会影响默认的 binder 配置。为了实现这一点，binder 配置可以将其 defaultCandidate 标志设置为 false：
>
> ```properties
> spring.cloud.stream.binders.<configurationName>.defaultCandidate=false
> ```
>
> 这表示该配置存在独立于默认 binder 配置过程之外。

以下示例展示了一个应用程序的典型配置，该应用程序连接到两个 RabbitMQ 代理服务器实例：

```yml
spring:
  cloud:
    stream:
      bindings:
        input:
          destination: thing1
          binder: rabbit1
        output:
          destination: thing2
          binder: rabbit2
      binders:
        rabbit1:
          type: rabbit
          environment:
            spring:
              rabbitmq:
                host: <host1>
        rabbit2:
          type: rabbit
          environment:
            spring:
              rabbitmq:
                host: <host2>
```

## Binding 的可视化和控制 <a href="#binding_visualization_control" id="binding_visualization_control"></a>

Spring Cloud Stream 支持通过 Actuator 端点进行绑定的可视化和控制。通过设置以下属性来启用绑定的 actuator 端点：

```properties
management.endpoints.web.exposure.include=bindings
```

一旦满足这些先决条件，当应用程序启动时，在日志中可以看到以下内容：

```less
: Mapped "{[/actuator/bindings/{name}],methods=[POST]. . .
: Mapped "{[/actuator/bindings],methods=[GET]. . .
: Mapped "{[/actuator/bindings/{name}],methods=[GET]. . .
```

要可视化当前的绑定，请访问以下 URL：

```properties
http://<host>:<port>/actuator/bindings
```
