# 编程模型



<figure><img src="../../../../.gitbook/assets/SCSt-with-binder.png" alt=""><figcaption></figcaption></figure>

To understand the programming model, you should be familiar with the following core concepts:

* **Destination Binders:** Components responsible to provide integration with the external messaging systems.
* **Bindings:** Bridge between the external messaging systems and application provided _Producers_ and _Consumers_ of messages (created by the Destination Binders).
* **Message:** The canonical data structure used by producers and consumers to communicate with Destination Binders (and thus other applications via external messaging systems).

## Destination Binders

Destination Binders are extension components of Spring Cloud Stream responsible for providing the necessary configuration and implementation to facilitate integration with external messaging systems. This integration is responsible for connectivity, delegation, and routing of messages to and from producers and consumers, data type conversion, invocation of the user code, and more.

## _Bindings_&#x20;

_Bindings_ provide a bridge between the external messaging system (e.g., queue, topic etc.) and application-provided _Producers_ and _Consumers_.

### **Functional binding names**

Unlike the explicit naming required by annotation-based support (legacy) used in the previous versions of spring-cloud-stream, the functional programming model defaults to a simple convention when it comes to binding names, thus greatly simplifying application configuration. Let’s look at the first example:

```java
@SpringBootApplication
public class SampleApplication {

	@Bean
	public Function<String, String> uppercase() {
	    return value -> value.toUpperCase();
	}
}
```

In the preceding example we have an application with a single function which acts as message handler. As a `Function` it has an input and output. The naming convention used to name input and output bindings is as follows:

* input - `<functionName> + -in- + <index>`
* output - `<functionName> + -out- + <index>`

The `in` and `out` corresponds to the type of binding (such as _input_ or _output_). The `index` is the index of the input or output binding. It is always 0 for typical single input/output function

So if for example you would want to map the input of this function to a remote destination (e.g., topic, queue etc) called "my-topic" you would do so with the following property:

```
spring.cloud.stream.bindings.uppercase-in-0.destination=my-topic
```

Note how `uppercase-in-0` is used as a segment in property name. The same goes for `uppercase-out-0`.

### **Descriptive Binding Names**

Some times to improve readability you may want to give your binding a more descriptive name (such as 'account', 'orders' etc). Another way of looking at it is you can map an _implicit binding name_ to an _explicit binding name_. And you can do it with `spring.cloud.stream.function.bindings.<binding-name>` property.

For example,

```
--spring.cloud.stream.function.bindings.uppercase-in-0=input
```

In the preceding example you mapped and effectively renamed uppercase-in-0 binding name to input. Now all configuration properties can refer to input binding name instead (e.g., --spring.cloud.stream.bindings.input.destination=my-topic).

> 建议完全避免使用它，特别是因为不使用它可以在binder目标和绑定名称之间提供明确的路径，例如 spring.cloud.stream.bindings.uppercase-in-0.destination=sample-topic。

### **Explicit binding creation**

Spring Cloud Stream allows you to define input and output bindings explicitly via `spring.cloud.stream.input-bindings` and `spring.cloud.stream.output-bindings` properties. Noticed the plural in the property names allowing you to define multiple bindings by simply using `;` as a delimiter.

## **Spring Cloud Function support**

To specify which functional bean to bind to the external destination(s) exposed by the bindings, you must provide `spring.cloud.function.definition` property.

> In the event you only have single bean of type `java.util.function.[Supplier/Function/Consumer]`, you can skip the `spring.cloud.function.definition` property, since such functional bean will be auto-discovered. However, it is considered best practice to use such property to avoid any confusion. Some time this auto-discovery can get in the way, since single bean of type `java.util.function.[Supplier/Function/Consumer]` could be there for purposes other then handling messages, yet being single it is auto-discovered and auto-bound. For these rare scenarios you can disable auto-discovery by providing `spring.cloud.stream.function.autodetect` property with value set to `false`.

`Function` and `Consumer` are pretty straightforward when it comes to how their invocation is triggered. They are triggered based on data (events) sent to the destination they are bound to.

### **Functional Composition**

Using functional programming model you can also benefit from functional composition where you can dynamically compose complex handlers from a set of simple functions. As an example let’s add the following function bean to the application defined above

```java
@Bean
public Function<String, String> wrapInQuotes() {
	return s -> "\"" + s + "\"";
}
```

and modify the `spring.cloud.function.definition` property to reflect your intention to compose a new function from both ‘toUpperCase’ and ‘wrapInQuotes’. To do so Spring Cloud Function relies on `|` (pipe) symbol. So, to finish our example our property will now look like this:

```java
spring.cloud.function.definition=toUpperCase|wrapInQuotes
```

The result of a composition is a single function which, as you may guess, could have a very long and rather cryptic name (e.g., `foo|bar|baz|xyz. . .`) presenting a great deal of inconvenience when it comes to other configuration properties. This is where _descriptive binding names_ feature described in [Functional binding names](https://docs.spring.io/spring-cloud-stream/docs/4.0.4-SNAPSHOT/reference/html/spring-cloud-stream.html#\_functional\_binding\_names) section can help.

For example, if we want to give our `toUpperCase|wrapInQuotes` a more descriptive name we can do so with the following property `spring.cloud.stream.function.bindings.toUpperCase|wrapInQuotes-in-0=quotedUpperCaseInput` allowing other configuration properties to refer to that binding name (e.g., `spring.cloud.stream.bindings.quotedUpperCaseInput.destination=myDestination`).

### **Batch Consumers**

When using a `MessageChannelBinder` that supports batch listeners, and the feature is enabled for the consumer binding, you can set `spring.cloud.stream.bindings.<binding-name>.consumer.batch-mode` to `true` to enable the entire batch of messages to be passed to the function in a `List`.

```java
@Bean
public Function<List<Person>, Person> findFirstPerson() {
    return persons -> persons.get(0);
}
```

### **Batch Producers**

You can also use the concept of batching on the producer side by returning a collection of Messages which effectively provides an inverse effect where each message in the collection will be sent individually by the binder.

Consider the following function:

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

Each message in the returned list will be sent individually resulting in four messages sent to output destination.

## **Sending arbitrary data to an output**

There are cases where the actual source of data may be coming from the external (foreign) system that is not a binder.

Spring Cloud Stream provides two mechanisms, so let’s look at them in more details.

```java
@SpringBootApplication
@Controller
public class WebSourceApplication {

	public static void main(String[] args) {
		SpringApplication.run(WebSourceApplication.class, 
		"--spring.cloud.stream.output-bindings=toStream");
	}

	@Autowired
	private StreamBridge streamBridge;

	@RequestMapping
	@ResponseStatus(HttpStatus.ACCEPTED)
	public void delegateToSupplier(@RequestBody String body) {
		System.out.println("Sending " + body);
		streamBridge.send("toStream", body);
	}
}
```

Here we autowire a `StreamBridge` bean which allows us to send data to an output binding effectively bridging non-stream application with spring-cloud-stream. Note that preceding example does not have any source functions defined leaving the framework with no trigger to create source bindings in advance, which would be typical for cases where configuration contains function beans. And that is fine, since `StreamBridge` will initiate creation of output bindings (as well as destination auto-provisioning if necessary) for non existing bindings on the first call to its `send(..)` operation caching it for subsequent reuse.&#x20;

And here you’re also benefiting from the dynamic features of `StreamBridge` where if `myBinding` doesn’t exist it will be created automatically and cached, otherwise existing binding will be used.

`streamBridge.send(..)` method takes an `Object` for data. This means you can send POJO or `Message` to it and it will go through the same routine when sending output as if it was from any Function or Supplier providing the same level of consistency as with functions. This means the output type conversion, partitioning etc are honored as if it was from the output produced by functions.

> Caching dynamic destinations (bindings) could result in memory leaks in the event there are many dynamic destinations. To have some level of control we provide a self-evicting caching mechanism for output bindings with default cache size of 10. This means that if your dynamic destination size goes above that number, there is a possibility that an existing binding will be evicted and thus would need to be recreated which could cause minor performance degradation. You can increase the cache size via `spring.cloud.stream.dynamic-destination-cache-size` property setting it to the desired value.

### **Using channel interceptors with StreamBridge**

Since `StreamBridge` uses a `MessageChannel` to establish the output binding, you can activate channel interceptors when sending data through `StreamBridge`. It is up to the application to decide which channel interceptors to apply on `StreamBridge`. Spring Cloud Stream does not inject all the channel interceptors detected into `StreamBridge` unless they are annoatated with `@GlobalChannelInterceptor(patterns = "*")`.

Let us assume that you have the following two different `StreamBridge` bindings in the application.

`streamBridge.send("foo-out-0", message);`

and

`streamBridge.send("bar-out-0", message);`

Now, if you want a channel interceptor applied on both the `StreamBridge` bindings, then you can declare the following `GlobalChannelInterceptor` bean.

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

if you don’t like the global approach above and want to have a dedicated interceptor for each binding, then you can do the following.

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

You have the flexibility to make the patterns more strict or customized to your business needs.

## Error Handling <a href="#spring-cloud-stream-overview-error-handling" id="spring-cloud-stream-overview-error-handling"></a>

Whenever Message handler (function) throws an exception, it is propagated back to the binder, at which point binder will make several attempts at re-trying the same message (3 by default) using `RetryTemplate` provided by the [Spring Retry](https://github.com/spring-projects/spring-retry) library. If retries are unsuccessful it is up to the error handling mechanism which may _drop_ the message, _re-queue_ the message for re-processing or _send the failed message to DLQ_.

Both Rabbit and Kafka support these concepts (especially DLQ).

### **Drop Failed Messages**

By default, the system provides error handlers. The first error handler will simply log error message. The second error handler is binder specific error handler which is responsible for handling error message in the context of a specific messaging system (e.g., send to DLQ). But since no additional error handling configuration was provided (in this current scenario) this handler will not do anything. So essentially after being logged, the message will be dropped.

### **Handle Error Messages**

The framework also exposes mechanism for you to provide custom error handler (i.e., to send notification or write to database, etc). You can do so by adding `Consumer` that is specifically designed to accept `ErrorMessage` which aside form all the information about the error (e.g., stack trace etc) contains the original message (the one that triggered the error).

```java
@Bean
public Consumer<ErrorMessage> myErrorHandler() {
	return v -> {
		// send SMS notification code
	};
}
```

To identify such consumer as an error handler all you need is to provide `error-handler-definition` property pointing to the function name - `spring.cloud.stream.bindings.<binding-name>.error-handler-definition=myErrorHandler`.

For example, for binding name `uppercase-in-0` the property would look like this:

```
spring.cloud.stream.bindings.uppercase-in-0.error-handler-definition=myErrorHandler
```

> If you want to have a single error handler for all function beans, you can use the standard spring-cloud-stream mechanism for defining default properties `spring.cloud.stream.default.error-handler-definition=myErrorHandler`

### **DLQ - Dead Letter Queue**

DLQ allows failed messages to be sent to a special destination: the _Dead Letter Queue_. When configured, failed messages are sent to this destination for subsequent re-processing or auditing and reconciliation.

Consider the following example:

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
	      });
		};
	}
}
```

Aside from some standard properties we also set the `auto-bind-dlq` to instruct the binder to create and configure DLQ destination for `uppercase-in-0` binding which corresponds to `uppercase` destination (see corresponding property), which results in an additional Rabbit queue named `uppercase.myGroup.dlq` (see Kafka documentation for Kafka specific DLQ properties).

Once configured, all failed messages are routed to this destination preserving the original message for further actions.

You can also facilitate immediate dispatch to DLQ (without re-tries) by setting `max-attempts` to '1'. For example,

```
spring.cloud.stream.bindings.uppercase-in-0.consumer.max-attempts=1
```

### **Retry Template**

The `RetryTemplate` is part of the [Spring Retry](https://github.com/spring-projects/spring-retry) library.we will mention the following consumer properties that are specifically related to the `RetryTemplate`:

* maxAttempts：
  * The number of attempts to process the message.
  * Default: 3.
* backOffInitialInterval
  * The backoff initial interval on retry.
  * Default 1000 milliseconds.
* backOffMaxInterval
  * The maximum backoff interval.
  * Default 10000 milliseconds.
* backOffMultiplier
  * The backoff multiplier.
  * Default 2.0.
* defaultRetryable
  * Whether exceptions thrown by the listener that are not listed in the `retryableExceptions` are retryable.
  * Default: `true`.
* retryableExceptions
  * A map of Throwable class names in the key and a boolean in the value. Specify those exceptions (and subclasses) that will or won’t be retried. Also see `defaultRetriable`. Example: `spring.cloud.stream.bindings.input.consumer.retryable-exceptions.java.lang.IllegalStateException=false`.
  * Default: empty.
*
