# Redis Template

与其他的 Spring Data 项目类似，Spring Data Redis 以模板的形式提供了较高等级的数据访问方案。**Spring Data Redis 提供了两个模板**：

*   <mark style="color:blue;">**RedisTemplate**</mark>

    **RedisTemplate 可以极大地简化 Redis 数据访问，能够让我们持久化各种类型的 key 和 value，并不局限于字节数组。**
*   <mark style="color:blue;">**StringRedisTemplate**</mark>

    在认识到 key 和 value 通常是 String 类型之后，**StringRedisTemplate 扩展了 RedisTemplate，只关注 String 类型。**

假设已经有了 RedisConnectionFactory，那么可以按照如下的方式构建 RedisTemplate：

```java
RedisConnectionFactory cf = ...;
RedisTemplate<String, Product> redis = new RedisTemplate<String, Product>();
redis.setConnectionFactory(cf);
```

如果你所使用的 value 和 key 都是 String 类型，那么可以考虑使用 StringRedisTemplate 来代替 RedisTemplate：

```java
RedisConnectionFactory cf = ...;
RedisTemplate redis = new StringRedisTemplate();
```

如果经常使用 RedisTemplate 或 StringRedisTemplate 的话，可以考虑将其配置为 bean，然后注入到需要的地方。

如下就是一个声明 RedisTemplate 的简单 @Bean 方法：

```java
@Bean
public RedisTemplate<String, Product> redisTemplate(RedisConnectionFactory cf) {
  RedisTemplate<String, Product> redis = new RedisTemplate<String, Product>();
  redis.setConnectionFactory(cf);
  return redis;
}
```

如下是声明 StringRedisTemplate bean 的 @Bean 方法：

```java
@Bean
public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory cf) {
  return new StringRedisTemplate(cf);
}
```
