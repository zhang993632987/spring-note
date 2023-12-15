# 连接到 Redis

Redis 连接工厂会生成到 Redis 数据库服务器的连接。Spring Data Redis 为四种 Redis 客户端实现提供了连接工厂：

* **JedisConnectionFactory**
* **JredisConnectionFactory**
* **LettuceConnectionFactory**
* **SrpConnectionFactory**

从 Spring Data Redis 的角度来看，这些连接工厂在适用性上都是相同的。

可以将连接工厂配置为 Spring 中的 bean：

```java
@Bean
public RedisConnectionFactory redisCF() {
  return new JedisConnectionFactory();
}
```

通过默认构造器创建的连接工厂会向 localhost 上的 6379 端口创建连接，并且没有密码。如果 Redis 服务器运行在其他的主机或端口 上，在创建连接工厂的时候，可以设置这些属性：

```java
@Bean
public RedisConnectionFactory redisCF() {
  JedisConnectionFactory cf = new JedisConnectionFactory();
  cf.setHostName("redis-server");
  cf.setPort(7379);
  return cf;
}
```
