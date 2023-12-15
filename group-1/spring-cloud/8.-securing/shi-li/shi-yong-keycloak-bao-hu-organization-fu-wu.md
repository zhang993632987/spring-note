# 使用Keycloak保护 organization 服务

一旦我们在Keycloak服务器中注册了客户端并设置了具有角色的各个用户帐户，我们就可以开始探索如何使用Spring Security和Keycloak Spring Boot适配器来保护资源。

<mark style="color:blue;">**尽管访问令牌的创建和管理是Keycloak服务器的职责，但在Spring应用中，确定哪些用户角色有权执行哪些操作是在各个服务中进行的。**</mark>为了设置受保护的资源，需要执行以下操作：

1. 将Spring Security和Keycloak 的JAR包添加到要保护的服务中。
2. 配置要保护的服务，以指向Keycloak服务器。
3. 定义能够谁能够访问服务。

## 将Spring Security和Keycloak 的JAR包添加到要保护的服务中

需要在 organization 服务的Maven配置文件中添加一些依赖项：

```xml
<dependencies>
    <!-- OAuth2 Client dependency -->
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-oauth2-client</artifactId>
    </dependency>
    <!-- OAuth2 Client dependency -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
</dependencies>
```

## 配置服务，以指向Keycloak服务器

一旦组织服务被设置为受保护的资源。每次对服务进行调用时，<mark style="color:blue;">**调用者必须包含HTTP标头Authentication，标头中包含一个Bearer访问令牌。受保护资源在收到令牌后，必须回调Keycloak服务器以认证令牌是否有效。**</mark>

{% code overflow="wrap" %}
```properties
spring:
  security:
      oauth2:
        client:
            registration:
              login-client:
                provider: keycloak
                client-id: learning
                client-secret: "BMF0NGhYC9Gk4nsjOXTiFc56rnPzHcuY"
                authorization-grant-type: authorization_code
                scope: openid,profile,roles,email
            provider:
              keycloak:
                issuer-uri: "http://192.168.10.110:8073/realms/learning-realm"
```
{% endcode %}

## 定义能够谁能够访问服务

<details>

<summary><mark style="color:purple;">OAuth2SecurityConfig</mark></summary>

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(jsr250Enabled = true)
public class OAuth2SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) 
            throws Exception {
        http.authorizeRequests().
                anyRequest().authenticated()
                .and()
                .csrf().disable()
                .oauth2ResourceServer(OAuth2ResourceServerConfigurer::jwt);
        return http.build();
    }
}
```

</details>

访问规则可以从粗粒度（任何经过身份认证的用户都可以访问整个服务）到细粒度（只有具有指定角色的应用程序可以访问，但只能通过DELETE访问此URL）的范围。

### 任何经过身份认证的用户都可以访问整个服务

```java
@Override
protected void configure(HttpSecurity http) 
        throws Exception {    
  super.configure(http);
  http.authorizeRequests()    
      .anyRequest().authenticated();
}
```

所有访问规则都在configure()方法内定义。我们将使用Spring传入的HttpSecurity类来定义我们的规则。在本例中，我们将限制对 organization 服务中的任何URL的访问，仅允许经过身份认证的用户。

请记住，<mark style="color:blue;">**在调用 organization 服务时，我们需要将授权类型设置为Bearer Token，并携带access\_token的值。**</mark>

### 通过特定角色保护服务

```java
@RestController
@RequestMapping(value="v1/organization")
public class OrganizationController {
    @Autowired
    private OrganizationService service;
    
    @RolesAllowed({ "ADMIN", "USER" })  
    @RequestMapping(value="/{organizationId}",method = RequestMethod.GET)
    public ResponseEntity<Organization> getOrganization(
           @PathVariable("organizationId") String organizationId) {
        return ResponseEntity.ok(service.findById(organizationId));
    }
    
    @RolesAllowed({ "ADMIN", "USER" })    
    @RequestMapping(value="/{organizationId}",method = RequestMethod.PUT)
    public void updateOrganization( @PathVariable("organizationId") 
                String id, @RequestBody Organization organization) {
        service.update(organization);
    }
    
    @RolesAllowed({ "ADMIN", "USER" })     
    @PostMapping
    public ResponseEntity<Organization>  saveOrganization(
                 @RequestBody Organization organization) {
        return ResponseEntity.ok(service.create(organization));
    }
    
    @RolesAllowed("ADMIN")     
    @DeleteMapping(value="/{organizationId}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteLicense(@PathVariable("organizationId") 
                                    String organizationId) {
        service.delete(organizationId);
    }    
}
```
