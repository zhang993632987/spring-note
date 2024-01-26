# Redis 事务

Redis 通过 multi、exec 和 discard 命令提供事务支持。这些操作在 RedisTemplate 上是可用的。然而，**不能保证 RedisTemplate 会在同一个连接中运行事务中的所有操作。**&#x20;

<mark style="color:blue;">**Spring Data Redis 为在同一个连接中执行多个操作（例如在使用 Redis 事务时）提供了 SessionCallback 接口。**</mark>以下示例使用了 multi 方法：

```java
//execute a transaction
List<Object> txResults = redisTemplate.execute(new SessionCallback<List<Object>>() {
  public List<Object> execute(RedisOperations operations) 
      throws DataAccessException {
    operations.multi();
    operations.opsForSet().add("key", "value1");

    // This will contain the results of all operations in the transaction
    return operations.exec();
  }
});
System.out.println("Number of items added to set: " + txResults.get(0));
```

在返回之前，RedisTemplate 使用其值、哈希键和哈希值的序列化器对 exec 的所有结果进行反序列化。此外，还有一个额外的 exec 方法，允许传递用于事务结果的自定义序列化器。

## **@Transactional**&#x20;

{% hint style="warning" %}
## <mark style="color:orange;">注意</mark>

**默认情况下，RedisTemplate 不参与 Spring 管理的事务。**

**如果希望在使用 @Transactional 或 TransactionTemplate 时，RedisTemplate 使用 Redis 事务，需要设置setEnableTransactionSupport(true)来显式启用事务支持。**
{% endhint %}

**启用事务支持将在 ThreadLocal 中绑定一个 RedisConnection 以供当前事务使用。**如果事务在没有错误的情况下完成，则通过 EXEC 提交 Redis 事务，否则通过 DISCARD 回滚。

<mark style="color:blue;">**Redis 事务是批处理导向的。在事务运行期间发出的命令会被排队，只有在提交事务时才会应用。**</mark>

Spring Data Redis 在事务中会区分只读和写命令：

* 只读命令（例如 KEYS）会被导向一个新的（非线程绑定的）RedisConnection 以允许读取。&#x20;
* 写命令由 RedisTemplate 排队，并在提交时应用。&#x20;

以下示例展示了如何配置事务管理：

```java
@Configuration
@EnableTransactionManagement                                 
public class RedisTxContextConfiguration {

  @Bean
  public StringRedisTemplate redisTemplate() {
    StringRedisTemplate template = new StringRedisTemplate(redisConnectionFactory());
    // explicitly enable transaction support
    template.setEnableTransactionSupport(true);              
    return template;
  }

  @Bean
  public RedisConnectionFactory redisConnectionFactory() {
    // jedis || Lettuce
  }

  @Bean
  public PlatformTransactionManager transactionManager() throws SQLException {
    return new DataSourceTransactionManager(dataSource());   
  }

  @Bean
  public DataSource dataSource() throws SQLException {
    // ...
  }
}
```

* @EnableTransactionManagement: 启动声明式事务支持。
* template.setEnableTransactionSupport(true): 配置 RedisTemplate 以使其参与 Spring 事务。
* 事务管理需要一个 PlatformTransactionManager。
  * **Spring Data Redis 并没有附带PlatformTransactionManager 的实现。**
  * 假设应用程序使用 JDBC，Spring Data Redis 可以通过使用现有的事务管理器来参与事务。

以下示例分别展示了使用约束：

```java
// 必须在与线程绑定的连接上执行。
template.opsForValue().set("thing1", "thing2");

// 读操作必须在自由的（非事务感知的）连接上运行。
template.keys("*");

// 由于在事务中设置的值不可见，返回null。
template.opsForValue().get("thing1");
```
