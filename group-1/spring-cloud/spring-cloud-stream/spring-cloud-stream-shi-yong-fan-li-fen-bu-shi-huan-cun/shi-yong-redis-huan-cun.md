# 使用Redis缓存

要在 licensing 服务中使用Redis，需要执行以下操作：

1. 配置许可服务以包含 Spring Data Redis 依赖项。
2. 配置到 Redis 的数据库连接。
3. 定义 Spring Data Redis 仓库，代码将使用它们与 Redis 的哈希类型进行交互。
4. 使用 Redis 和 licensing 服务来存储和读取组织数据。

## 添加依赖

```xml
<dependency>
   <groupId>org.springframework.data</groupId>
   <artifactId>spring-data-redis</artifactId>
</dependency>
<dependency>
   <groupId>redis.clients</groupId>
   <artifactId>jedis</artifactId>
</dependency>
```

## 配置Redis连接

```yaml
spring:
  redis:
    host: 192.168.10.110
    port: 6379
    jedis:
      pool:
        enabled: true
        min-idle: 8
        max-active: 8
        max-wait: 5s
```

## 定义 Spring Data Redis 仓库

对于 licensing 服务，将定义两个文件：

*   第一个文件是一个Java接口，它将被注入到任何需要访问Redis的 licensing 服务类中。

    ```java
    @Repository
    public interface OrganizationRedisRepository extends
         CrudRepository<Organization,String>{
    }
    ```

    通过扩展**CrudRepository**，OrganizationRedisRepository包含了用于在Redis中存储和检索数据的所有CRUD（创建、读取、更新、删除）逻辑。
*   第二个文件是一个模型类。这个类是一个POJO，包含了将要存储在Redis缓存中的数据。

    <pre class="language-java"><code class="lang-java"><strong>@Getter @Setter @ToString
    </strong>@RedisHash("organization")   
    public class Organization {
        @Id
        String id;
        String name;
        String contactName;
        String contactEmail;
        String contactPhone;  
    }
    </code></pre>

    注意：**Redis服务器可以包含多个哈希结构，因此，在与Redis的每次交互中，需要告诉Redis操作的数据结构的名称。**

## 存储和读取组织数据

每当 licensing 服务需要组织数据时，它会检查Redis缓存，如果缓存未命中才访问 organization 服务：

{% code overflow="wrap" lineNumbers="true" %}
```java
@Slf4j
@Component
public class OrganizationRestTemplateClient {

    private RestTemplate restTemplate;

    private OrganizationRedisRepository redisRepository;

    private Organization checkRedisCache(String organizationId) {
        try {
            return redisRepository
                    .findById(organizationId)
                    .orElse(null);
        } catch (Exception ex) {
            log.error("Error encountered while trying to retrieve organization{} check Redis Cache. Exception {}",
                    organizationId, ex);
            return null;
        }
    }

    private void cacheOrganizationObject(Organization organization) {
        try {
            redisRepository.save(organization);
        } catch (Exception ex) {
            log.error("Unable to cache organization {} in Redis.Exception {}",
                    organization.getId(), ex);
        }
    }

    public Organization getOrganization(String organizationId) {
        log.debug("In Licensing Service.getOrganization: {}",
                UserContextHolder.getContext().getCorrelationId());
        Organization organization = checkRedisCache(organizationId);
        if (organization != null) {
            log.debug("I have successfully retrieved an organization {} from the redis cache: {}",
                    organizationId, organization);
            return organization;
        }
        log.debug("Unable to locate organization from the redis cache:{}.", organizationId);

        organization = restTemplate.getForObject(
                "http://{applicationId}/v1/organization/{organizationId}",
                Organization.class,
                "organization-service",
                organizationId);
        if (organization != null) {
            cacheOrganizationObject(organization);
        }
        return organization;
    }

    @Autowired
    public void setRestTemplate(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    @Autowired
    public void setRedisRepository(OrganizationRedisRepository redisRepository) {
        this.redisRepository = redisRepository;
    }
}
```
{% endcode %}

* getOrganization() 方法是进行组织服务调用的地方。在进行实际的 REST 调用之前，需要先使用 checkRedisCache() 方法从 Redis 中检索与调用相关联的 Organization 对象。
* 如果想要的组织对象不在 Redis 中，代码将返回一个 null 值。
  * 如果 checkRedisCache() 方法返回 null 值，代码将调用组织服务的 REST 端点以检索所需的组织记录。
  * 如果 organization 服务返回一个组织对象，返回的组织对象将使用 cacheOrganizationObject() 方法进行缓存。

> ## 注意：
>
> **在与缓存交互时要特别注意异常处理。**
>
> 为了增加弹性，当无法与Redis服务器通信时，我们并未让调用失败。相反，我们记录异常并允许调用到达组织服务。**在这种特定情况下，缓存旨在帮助提高性能，而缓存服务器的缺失并不影响调用的成功。**
