# Using Redis to cache lookups

To useRedis in the licensing service, we need to do the following:

1. Configure the licensing service to include the Spring Data Redis dependencies.&#x20;
2. Construct a database connection to Redis.
3. Define the Spring Data Redis repositories that our code will use to interact with a Redis hash.
4. Use Redis and the licensing service to store and read organization data.

## CONFIGURING THE LICENSING SERVICE WITH SPRING DATA REDIS DEPENDENCIES

The first thing we need to do is include the spring-data-redis dependencies, along with jedis into the licensing service’s pom.xml file. The next listing shows these dependencies.

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

## CONSTRUCTING THE DATABASE CONNECTION TO A REDIS SERVER

```
// Some code
```

Once we have a connection to Redis, we’ll use that connection to create a Spring RedisTemplate object.

```java
@Bean   
public RedisTemplate<String, Object> redisTemplate() {
   RedisTemplate<String, Object> template = new RedisTemplate<>();
   template.setConnectionFactory(jedisConnectionFactory());
   return template;
}
```

## DEFINING THE SPRING DATA REDIS REPOSITORIES

For thelicensing service, we’ll define two files for our Redis repository.&#x20;

The first file is a Java interface that will be injected into any of the licensing service classes that need to access Redis.

```java
@Repository
public interface OrganizationRedisRepository extends
     CrudRepository<Organization,String>{
}
```

By extending from CrudRepository, OrganizationRedisRepository contains all the CRUD (Create, Read, Update, Delete) logic used for storing and retrieving data from Redis (in this case).&#x20;

The second file is the model we’ll use for our repository. This class is a POJO containing the data that we’ll store in our Redis cache.

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

One important thing to note from the code is that a Redis server can contain multiple hashes and data structures within it. We therefore need to tell Redis the name of the data structure we want to perform the operation against in every interaction with Redis.

## USING REDIS AND THE LICENSING SERVICE TO STORE AND READ ORGANIZATION DATA

Every time the licensing service needs the organization data, it checks the Redis cache before calling the organization service. You’ll find the logic for doing this in the OrganizationRestTemplateClient class.

```java
@Component
public class OrganizationRestTemplateClient {
    @Autowired
    RestTemplate restTemplate;
    @Autowired
    OrganizationRedisRepository redisRepository;    
    private static final Logger logger =
         LoggerFactory.getLogger(OrganizationRestTemplateClient.class);
         
    private Organization checkRedisCache(String organizationId) {
        try {
           return redisRepository
                    .findById(organizationId)
                    .orElse(null);   
        }catch (Exception ex){
           logger.error("Error encountered while trying to retrieve 
            organization{} check Redis Cache.  Exception {}", 
            organizationId, ex);
           return null;
        }
    }
    
    private void cacheOrganizationObject(Organization organization) {
        try {
           redisRepository.save(organization);    
        }catch (Exception ex){
           logger.error("Unable to cache organization {} in 
            Redis. Exception {}",
            organization.getId(), ex);
        }
    }
    
    public Organization getOrganization(String organizationId){
        logger.debug("In Licensing Service.getOrganization: {}", 
            UserContext.getCorrelationId());
        Organization organization = checkRedisCache(organizationId);
        if (organization != null){    
          logger.debug("I have successfully retrieved an organization
            {} from the redis cache: {}", organizationId,
            organization);
        return organization;
        }
        logger.debug("Unable to locate organization from the 
            redis cache: {}.",organizationId);
        ResponseEntity<Organization> restExchange =
           restTemplate.exchange(
            "http://gateway:8072/organization/v1/organization/
            {organizationId}",HttpMethod.GET,
            null, Organization.class, organizationId);
        organization = restExchange.getBody();
        if (organization != null) {
          cacheOrganizationObject(organization);
        }
        return restExchange.getBody();
   }
}    
```

The getOrganization() method is where the call to the organization service takes place. Before we make the actual REST call, we need to retrieve the Organization object associated with the call from Redis using the checkRedisCache() method.

If the organization object in question is not in Redis, the code returns a null value. If a null value is returned from the checkRedisCache() method, the code invokes the organization service’s REST endpoint to retrieve the desired organization record. If the organization service returns an organization, the returned organization object is cached using the cacheOrganizationObject() method.

> ## 注意：
>
> Pay close attention to exception handling when interacting with the cache. To increase resiliency, we never let the entire call fail if we cannot communicate with the Redis server. Instead, we log the exception and let the call through to the organization service. In this particular case, caching is meant to help improve performance, and the absence of the caching server shouldn’t impact the success of the call.
