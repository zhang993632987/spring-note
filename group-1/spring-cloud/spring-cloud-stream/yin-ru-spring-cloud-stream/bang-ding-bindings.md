# 绑定（Bindings）

Bindings 在外部消息系统（例如，队列、主题等）和应用提供的生产者和消费者之间建立了桥梁。

## 绑定名称

### 函数式绑定名称

与之前版本的 spring-cloud-stream 使用的基于注解的方式需要显式的命名不同，函数式编程模型在绑定名称方面采用了一种简单的约定，从而大大简化了应用程序配置。

```java
@SpringBootApplication
public class SampleApplication {

	@Bean
	public Function<String, String> uppercase() {
	    return value -> value.toUpperCase();
	}
}
```

上面的例子中定义了一个名为 uppercase 的函数，该函数充当消息处理程序。作为一个函数，它有一个输入和一个输出。用于命名输入和输出绑定的命名约定如下：

* **input - \<functionName> + -in- + \<index>**
* **output - \<functionName> + -out- + \<index>**

其中：

* **in** 和 **out**对应于绑定的类型（输入或输出）
* **\<index>**是输入或输出绑定的索引。对于典型的单输入/输出函数，它始终为0。

因此，如果想将此函数的输入映射到名为"my-topic"的远程目标，可以使用以下属性：

```properties
spring.cloud.stream.bindings.uppercase-in-0.destination=my-topic
```

### 描述性的绑定名称

有时为了提高可读性，可能希望为绑定指定一个更具描述性的名称。此时，可以使用**spring.cloud.stream.function.bindings.\<binding-name>**属性来实现。例如：

```properties
spring.cloud.stream.function.bindings.uppercase-in-0=input
```

上面的例子将uppercase-in-0绑定重命名为了input。之后，所有的配置属性都可以引用input。例如：

```properties
spring.cloud.stream.bindings.input.destination=my-topic
```

> 建议完全避免使用它，特别是因为不使用它可以在binder目标和绑定名称之间提供明确的路径，例如 spring.cloud.stream.bindings.uppercase-in-0.destination=sample-topic。

### 显式绑定

通过 **spring.cloud.stream.input-bindings** 和 **spring.cloud.stream.output-bindings** 属性可以显式定义输入和输出绑定。

> **通过使用分号 ; 作为分隔符可以定义多个绑定。**

## **指定函数 Bean**

要指定绑定的函数 bean，必须提供 **spring.cloud.function.definition** 属性。

> 如果只有一种类型为 java.util.function.\[Supplier/Function/Consumer] 的单个 bean，则可以省略 spring.cloud.function.definition 属性，因为可以自动发现该 bean。但是，**为了避免混淆，最好添加此属性。**
>
> 有时，自动发现可能会不符合需求，因为类型为 java.util.function.\[Supplier/Function/Consumer] 的单个 bean 可能出于处理消息以外的其他目的，但由于是单个 bean，它将被自动发现和自动绑定。对于这种情况，可以通过将 **spring.cloud.stream.function.autodetect** 设置为 false 来禁用自动发现。

### 函数组合

使用函数式编程模型，还可以从函数组合中受益，**可以动态地将一组简单的函数组合成复杂的处理程序**。

例如，在应用程序添加以下函数 bean：

```java
@Bean
public Function<String, String> wrapInQuotes() {
	return s -> "\"" + s + "\"";
}
```

修改 **spring.cloud.function.definition** 属性，从 'toUpperCase' 和 'wrapInQuotes' 中组合出一个新函数。为此，Spring Cloud Function 依赖 **|（管道）符号：**

```properties
spring.cloud.function.definition=toUpperCase|wrapInQuotes
```

**组合得到的结果是一个单一的函数，该函数有一个非常长且相当晦涩的名称**（如，foo|bar|baz|xyz...），在处理其他配置属性时可能会带来很大的不便。

可以使用给函数一个[#miao-shu-xing-de-bang-ding-ming-cheng](bang-ding-bindings.md#miao-shu-xing-de-bang-ding-ming-cheng "mention") 来消除这种不便。例如，如果要为 toUpperCase|wrapInQuotes 给予一个更具描述性的名称，我们可以通过以下属性来实现，从而使其他配置属性可以引用该绑定名称：

{% code overflow="wrap" %}
```properties
spring.cloud.stream.function.bindings.toUpperCase|wrapInQuotes-in-0=quotedUpperCaseInput
```
{% endcode %}

### 批量消费者

要使用批量消费者，可以通过将 **spring.cloud.stream.bindings.\<binding-name>.consumer.batch-mode** 属性设置为 true 来配置消费者的批处理模式。这告诉Spring Cloud Stream消费者应该以批处理模式运行，以便整个消息批次被传递给函数，形成一个List。

```java
@Bean
public Function<List<Person>, Person> findFirstPerson() {
    return persons -> persons.get(0);
}
```

**在批处理模式下操作时，消费者会接收到一组消息，使应用程序能够批量处理它们，而不是逐个处理。**这在处理单个消息效率较低的情况下特别有益。

### 批量生产者

还可以在生产者端使用批处理的概念，方法是返回一个Message集合，这实际上提供了一种反向效果，即**集合中的每个消息将由binder单独发送。**

考虑以下函数：

```java
@Bean
public Function<String, List<Message<String>>> batch() {
	return p -> {
		List<Message<String>> list = new ArrayList<>();
		list.add(MessageBuilder.withPayload(p + ":1").build());
		list.add(MessageBuilder.withPayload(p + ":2").build());
		list.add(MessageBuilder.withPayload(p + ":3").build());
		list.add(MessageBuilder.withPayload(p + ":4").build());
		return list;
	};
}
```

当将此函数配置为 Spring Cloud Stream 生产者时，**每个消息都将作为单独的消息发送到输出目标。**

## 将任意数据发送到输出

```java
@SpringBootApplication
@Controller
public class WebSourceApplication {

	public static void main(String[] args) {
		SpringApplication.run(WebSourceApplication.class);
	}

	@Autowired
	private StreamBridge streamBridge;

	@RequestMapping
	@ResponseStatus(HttpStatus.ACCEPTED)
	public void delegateToSupplier(@RequestBody String body) {
		System.out.println("Sending " + body);
		streamBridge.send("myStream", body);
	}
}
```

上述代码中注入了一个 StreamBridge bean，该 bean 负责将数据发送到输出绑定。请注意，前面的例子没有定义任何函数，这意味着没有提前创建绑定。**在这种情况下，StreamBridge 将在第一次调用其 send(..) 方法时创建输出绑定，并将其缓存以便后续重用。**

> **StreamBridge 的动态特性：**如果指定的 binding 不存在，它将被自动创建和缓存，否则将使用现有绑定。

> **缓存动态目标（绑定）可能导致内存泄漏，特别是在存在许多动态目标的情况下。**
>
> 为了实现一定程度的控制，Spring Cloud Stream 提供了一个自动逐出的缓存机制，默认缓存大小为 10。这意味着，如果动态目标数量超过该数值，可能会逐出现有的绑定，从而需要重新创建绑定，这可能导致轻微的性能下降。
>
> 可以通过将 **spring.cloud.stream.dynamic-destination-cache-size** 属性设置为所需的值来增加缓存大小。

### 使用 StreamBridge 的通道拦截器

由于 StreamBridge 使用 MessageChannel 来建立输出绑定，因此可以在通过 StreamBridge 发送数据时激活通道拦截器。

应用程序可以决定在 StreamBridge 上应用哪些通道拦截器。**Spring Cloud Stream 不会将检测到的所有通道拦截器注入到 StreamBridge 中，除非它们被标记为 @GlobalChannelInterceptor(patterns = "\*")。**

假设应用程序中有以下两个不同的 StreamBridge 绑定：

```java
streamBridge.send("foo-out-0", message);
streamBridge.send("bar-out-0", message);
```

如果希望在这两个 StreamBridge 绑定上都应用通道拦截器，可以定义如下的 GlobalChannelInterceptor bean。

```java
@Bean
@GlobalChannelInterceptor(patterns = "*")
public ChannelInterceptor customInterceptor() {
    return new ChannelInterceptor() {
        @Override
        public Message<?> preSend(Message<?> message, MessageChannel channel) {
            ...
        }
    };
}
```

如果不喜欢上述的全局方法，而是想为每个绑定使用专用的拦截器，那么可以采取以下方式：

```java
@Bean
@GlobalChannelInterceptor(patterns = "foo-*")
public ChannelInterceptor fooInterceptor() {
    return new ChannelInterceptor() {
        @Override
        public Message<?> preSend(Message<?> message, MessageChannel channel) {
            ...
        }
    };
}
```

## 错误处理 <a href="#spring-cloud-stream-overview-error-handling" id="spring-cloud-stream-overview-error-handling"></a>

当消息处理器（函数）抛出异常时，异常都会传播回绑定器，此时绑定器将使用 Spring Retry 库提供的 RetryTemplate 进行重试（默认为3次）。如果重试不成功，则消息将交由**错误处理机制**进行处理，该机制可能丢弃消息、重新将消息排队以供重新处理，或将失败的消息发送到死信队列（DLQ）。

> Rabbit 和 Kafka 都支持这些概念（尤其是死信队列 DLQ）。

### 放弃失败的消息

默认情况下，系统提供了多个错误处理程序：

* 第一个错误处理程序，只是简单地将错误消息记录到日志中；
* 第二个错误处理程序，当发生错误时，它负责以特定于消息系统的方式来处理错误消息。但是，由于在当前情况下没有提供额外的错误处理配置，因此此处理程序将不执行任何操作。

因此，错误消息在被记录到日志后，将被直接放弃。

### **自定义错误处理**

该框架提供了一种机制来提供自定义的错误处理程序（例如，发送通知或写入数据库等）。**可以通过添加专门设计用于接受 ErrorMessage 的 Consumer 来实现这一点**，其中 ErrorMessage 包含关于错误的所有信息（例如，堆栈跟踪等），以及触发错误的原始消息。

```java
@Bean
public Consumer<ErrorMessage> myErrorHandler() {
	return v -> {
		// send SMS notification code
	};
}
```

要将此类消费者标识为错误处理程序，只需提供 error-handler-definition 属性，该属性指向函数名称：

```properties
spring.cloud.stream.bindings.<binding-name>.error-handler-definition=myErrorHandler
```

**例如，对于名为 uppercase-in-0 的绑定，属性如下：**

```properties
spring.cloud.stream.bindings.uppercase-in-0.error-handler-definition=myErrorHandler
```

> 如果希望为所有函数 bean 使用单个错误处理程序，可以使用标准的 spring-cloud-stream 机制来定义默认属性：
>
> ```properties
> spring.cloud.stream.default.error-handler-definition=myErrorHandler
> ```

### **DLQ - Dead Letter Queue**

**DLQ（死信队列）允许将失败的消息发送到一个特殊的目的地，即死信队列**。配置后，失败的消息将发送到该目的地，以供后续重新处理、审计和协调。

考虑以下示例：

```java
@SpringBootApplication
public class SimpleStreamApplication {

	public static void main(String[] args) throws Exception {
		SpringApplication.run(SimpleStreamApplication.class,
		  "--spring.cloud.function.definition=uppercase",
		  "--spring.cloud.stream.bindings.uppercase-in-0.destination=uppercase",
		  "--spring.cloud.stream.bindings.uppercase-in-0.group=myGroup",
		  "--spring.cloud.stream.rabbit.bindings.uppercase-in-0.consumer.auto-bind-dlq=true"
		);
	}

	@Bean
	public Function<Person, Person> uppercase() {
		return personIn -> {
		   throw new RuntimeException("intentional");
		};
	}
}
```

除了一些标准属性外，还设置了 **auto-bind-dlq**，这是为了告诉绑定器为 uppercase 目的地（与 uppercase-in-0 绑定对应）创建并配置 DLQ（死信队列）。具体而言，这将导致一个额外的 Rabbit 队列，其名称为 uppercase.myGroup.dlq（有关 Kafka 特定的 DLQ 属性，请查阅 Kafka 文档）。

一旦配置完成，所有失败的消息都将被路由到该目的地，以便进行后续的重新处理、审计和协调。

还可以通过将 **max-attempts** 设置为 '1'，实现直接发送到 DLQ（无需重试）。例如：

```properties
spring.cloud.stream.bindings.uppercase-in-0.consumer.max-attempts=1
```

### **Retry Template**

RetryTemplate 是 Spring Retry 库的一部分。以下是与 RetryTemplate 相关的一些消费者属性：

* maxAttempts：处理消息的尝试次数。 默认值：3。
* backOffInitialInterval：重试时的初始间隔。 默认值：1000 毫秒。
* backOffMaxInterval：最大的重试间隔。 默认值：10000 毫秒。
* backOffMultiplier：重试间隔的倍增因子。 默认值：2.0。
* defaultRetryable：监听器抛出的未在 retryableExceptions 中列出的异常是否可重试。 默认值：true。
* retryableExceptions：一个以 Throwable 类名为键、布尔值为值的映射。指定那些异常（及其子类）将或不将被重试。示例：spring.cloud.stream.bindings.input.consumer.retryableExceptions.java.lang.IllegalStateException=false。 默认值：空。

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
