# 传递访问令牌

下图展示了用户令牌是如何通过网关、licensing 服务，然后传递到 organization 服务的：

<figure><img src="../../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

1. 用户已经通过Keycloak服务器进行了身份认证，并调用了O-stock Web应用程序。
   * 用户的访问令牌存储在用户的会话中。
   * O-stock Web应用程序需要检索一些许可数据并调用 licensing 服务的REST端点。
   * 在调用许可REST端点的过程中，O-stock Web应用程序通过HTTP Authentication 标头添加了访问令牌。
2. 服务网关查找 licensing 服务端点，然后将调用转发到 licensing 服务的其中一个服务器。<mark style="color:blue;">**服务网关复制传入调用的HTTP Authentication 标头，并确保将HTTP标头转发到新的端点。**</mark>
3. licensing 服务接收传入的调用。
   1. 由于 licensing 服务是受保护资源，licensing 服务将使用Keycloak服务器验证令牌，然后检查用户的角色以获取适当的权限。
   2. <mark style="color:blue;">**licensing 服务调用 organization 服务。在执行该操作时，licensing 服务需要将用户的访问令牌传播到organization 服务。**</mark>
4. 当organization 服务接收到调用时，它获取HTTP Authentication 标头并使用Keycloak服务器验证令牌。

## 修改服务网关配置

首先需要修改网关以便将访问令牌传递给 licensing服务。<mark style="color:blue;">**默认情况下，网关不会将诸如 Cookie、Set-cookie和Authorization之类的敏感HTTP标头转发到下游服务。**</mark>为了允许HTTP Authorization 标头的传播，我们需要在Spring Cloud Config存储库中的gateway-server.yml配置文件中添加以下默认过滤器：

```yaml
spring:
  cloud:
    gateway:
      default-filters:
        - RemoveRequestHeader=Cookie,Set-Cookie
```

**这个配置表示一个敏感标头的黑名单，网关将阻止这些标头传播到下游服务。在RemoveRequestHeader列表中不包含 Authorization 值意味着网关将允许该标头传递。如果我们不设置这个配置属性，网关会自动阻止所有三个值（Cookie、Set-Cookie和Authorization）的传播。**

## 修改 Licensing 服务

接下来，我们需要配置 licensing 服务以包含Keycloak和Spring Security的依赖项，并设置服务所需的授权规则。最后，我们需要将Keycloak属性添加到配置服务器中的应用程序属性文件。

### 配置与 organization 服务一样

{% content-ref url="shi-yong-keycloak-bao-hu-organization-fu-wu.md" %}
[shi-yong-keycloak-bao-hu-organization-fu-wu.md](shi-yong-keycloak-bao-hu-organization-fu-wu.md)
{% endcontent-ref %}

### RestTemplate

**如果没有Spring Security，我们将不得不编写一个servlet过滤器来获取传入许可服务调用的HTTP标头，然后将其添加到 licensing 服务中的每个出站服务调用中。**

<mark style="color:blue;">**Keycloak提供了一个新的支持这些调用的REST模板类，名为KeycloakRestTemplate。**</mark>要使用这个类，只需要将它公开为一个bean，然后利用 Spring 的自动注入即可。



<details>

<summary></summary>



</details>
