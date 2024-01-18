# Spring Cloud Stream 消费avro数据

## 问题 1：消息未序列化成 POJO

如果未将 specific.avro.reader 设置为 true，KafkaConsumer 只会将二进制流序列化成 GenericData$Record 对象，而不是指定的 POJO。

在这种情况下，GenericData$Record 对象在传入 consumer 函数时，会报 ClassCastException 异常！

{% code overflow="wrap" lineNumbers="true" %}
```log
Caused by: java.lang.ClassCastException: class org.apache.avro.generic.GenericData$Record cannot be cast to class com.study.organization.model.OrganizationChangeModel (org.apache.avro.generic.GenericData$Record is in unnamed module of loader 'app'; com.study.organization.model.OrganizationChangeModel is in unnamed module of loader org.springframework.boot.devtools.restart.classloader.RestartClassLoader @e49355b)
	at org.springframework.cloud.function.context.catalog.SimpleFunctionRegistry$FunctionInvocationWrapper.invokeConsumer(SimpleFunctionRegistry.java:1029) ~[spring-cloud-function-context-4.0.5.jar:4.0.5]
	at org.springframework.cloud.function.context.catalog.SimpleFunctionRegistry$FunctionInvocationWrapper.doApply(SimpleFunctionRegistry.java:731) ~[spring-cloud-function-context-4.0.5.jar:4.0.5]
	at org.springframework.cloud.function.context.catalog.SimpleFunctionRegistry$FunctionInvocationWrapper.apply(SimpleFunctionRegistry.java:577) ~[spring-cloud-function-context-4.0.5.jar:4.0.5]
	at org.springframework.cloud.stream.function.PartitionAwareFunctionWrapper.apply(PartitionAwareFunctionWrapper.java:92) ~[spring-cloud-stream-4.0.4.jar:4.0.4]
	at org.springframework.cloud.stream.function.FunctionConfiguration$FunctionWrapper.apply(FunctionConfiguration.java:832) ~[spring-cloud-stream-4.0.4.jar:4.0.4]
	at org.springframework.cloud.stream.function.FunctionConfiguration$FunctionToDestinationBinder$1.handleMessageInternal(FunctionConfiguration.java:661) ~[spring-cloud-stream-4.0.4.jar:4.0.4]
	at org.springframework.integration.handler.AbstractMessageHandler.doHandleMessage(AbstractMessageHandler.java:105) ~[spring-integration-core-6.1.5.jar:6.1.5]
	... 42 common frames omitted
```
{% endcode %}

**将 specific.avro.reader 设置为 true 可解决上述问题。**

## 问题 2：与 Spring Boot DevTools 冲突

Spring Boot DevTools 利用两个类加载器来实现热加载功能，对于项目中的类会使用 restart 类加载进行加载，但是 avro 序列化的 jar 包会使用 app 类加载器进行加载，也会导致 ClassCastException 异常

{% code overflow="wrap" lineNumbers="true" %}
```log
Caused by: java.lang.ClassCastException: class com.study.organization.model.OrganizationChangeModel cannot be cast to class com.study.organization.model.OrganizationChangeModel (com.study.organization.model.OrganizationChangeModel is in unnamed module of loader 'app'; com.study.organization.model.OrganizationChangeModel is in unnamed module of loader org.springframework.boot.devtools.restart.classloader.RestartClassLoader @e49355b)
	at org.springframework.cloud.function.context.catalog.SimpleFunctionRegistry$FunctionInvocationWrapper.invokeConsumer(SimpleFunctionRegistry.java:1029) ~[spring-cloud-function-context-4.0.5.jar:4.0.5]
	at org.springframework.cloud.function.context.catalog.SimpleFunctionRegistry$FunctionInvocationWrapper.doApply(SimpleFunctionRegistry.java:731) ~[spring-cloud-function-context-4.0.5.jar:4.0.5]
	at org.springframework.cloud.function.context.catalog.SimpleFunctionRegistry$FunctionInvocationWrapper.apply(SimpleFunctionRegistry.java:577) ~[spring-cloud-function-context-4.0.5.jar:4.0.5]
	at org.springframework.cloud.stream.function.PartitionAwareFunctionWrapper.apply(PartitionAwareFunctionWrapper.java:92) ~[spring-cloud-stream-4.0.4.jar:4.0.4]
	at org.springframework.cloud.stream.function.FunctionConfiguration$FunctionWrapper.apply(FunctionConfiguration.java:832) ~[spring-cloud-stream-4.0.4.jar:4.0.4]
	at org.springframework.cloud.stream.function.FunctionConfiguration$FunctionToDestinationBinder$1.handleMessageInternal(FunctionConfiguration.java:661) ~[spring-cloud-stream-4.0.4.jar:4.0.4]
	at org.springframework.integration.handler.AbstractMessageHandler.doHandleMessage(AbstractMessageHandler.java:105) ~[spring-integration-core-6.1.5.jar:6.1.5]
	... 42 common frames omitted
```
{% endcode %}

**只能去掉 spring-boot-devtools 依赖包，才能解决上述问题。**
