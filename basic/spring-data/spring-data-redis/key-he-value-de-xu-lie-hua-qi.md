# key 和 value 的序列化器

当某个条目保存到 Redis key-value 存储的时候，key 和 value 都会使用 Redis 的序列化器（serializer）进行序列化。Spring Data Redis 提供了多个这样的序列化器，包括：

* **GenericToStringSerializer**：使用 Spring 转换服务进行序列化；
* **JacksonJsonRedisSerializer**：使用 Jackson 1，将对象序列化为 JSON；
* **Jackson2JsonRedisSerializer**：使用 Jackson 2，将对象序列化为JSON；
* **JdkSerializationRedisSerializer**：使用 Java 序列化；
* **OxmSerializer**：使用 Spring O/X 映射的编排器和解排器 （marshaler 和 unmarshaler）实现序列化，用于 XML 序列化；
* **StringRedisSerializer**：序列化 String 类型的 key 和 value。

**这些序列化器都实现了 RedisSerializer 接口**，如果其中没有符合需求的序列化器，还可以自行创建。

**默认情况下：**

* <mark style="color:blue;">**RedisTemplate 会使用 JdkSerializationRedisSerializer**</mark>，这意味着 key 和 value 都会通过 Java 进行序列化。
* <mark style="color:blue;">**StringRedisTemplate 默认会使用 StringRedisSerializer**</mark>，它实际上就是实现 String 与 byte 数组之间的相互转换。

假设当使用 RedisTemplate 的时候，我们希望将 Product 类型的 value 序列化为 JSON，而 key 是 String 类型。**RedisTemplate 的 setKeySerializer() 和 setValueSerializer() 方法**就需要如下所示：

```java
@Bean
public RedisTemplate<String, Product> redisTemplate(RedisConnectionFactory cf) {
  RedisTemplate<String, Product> redis = new RedisTemplate<String, Product>();
  redis.setConnectionFactory(cf);
  redis.setKeySerializer(new StringRedisSerializer());
  redis.setValueSerializer(new Jackson2JsonRedisSerializer<Product>(Product.class));
  return redis;
}
```
