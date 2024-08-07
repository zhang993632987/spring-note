# 保护敏感配置信息

在默认情况下，Spring Cloud Config 服务器端在应用程序配置文件中以纯文本格式存储所有属性，包括像数据库凭据这样的敏感信息。

Spring Cloud Config 可以让我们轻松加密敏感属性。Spring Cloud Config 支持使用<mark style="color:blue;">**对称加密密钥**</mark>和<mark style="color:blue;">**非对称加密密钥**</mark>。

## 1. 创建对称加密密钥

对于 Spring Cloud Config 服务器端，对称加密密钥是一个由字母组成的字符串，你可以选择在 Config 服务器端的 <mark style="color:blue;">**application.yml**</mark> 文件中设置它，也可以选择通过操作系统环境变量 <mark style="color:blue;">**ENCRYPT\_KEY**</mark> 将它传递给服务。

{% hint style="info" %}
<mark style="color:blue;">**对称密钥的长度应该是 12 个或更多个字符，最好是一个随机的字符集。**</mark>
{% endhint %}

```yaml
encrypt:
  key: 123456789abcdef
```

## 2. 加密/解密

在启动 Spring Cloud Config 实例时，Spring Cloud Config 将检测到，环境变量 ENCRYPT\_KEY 或 application.yml 文件中的属性已设置，并自动将两个新端点 <mark style="color:blue;">**/encrypt**</mark> 和 <mark style="color:blue;">**/decrypt**</mark> 添加到 Spring Cloud Config 服务。

{% hint style="info" %}
**在调用 /encrypt 或 /decrypt 端点时，需要确保对这些端点进行 **<mark style="color:blue;">**POST**</mark>** 请求。**

<img src="../../../.gitbook/assets/image (10) (1).png" alt="" data-size="original">    ![](<../../../.gitbook/assets/image (12).png>)
{% endhint %}

## 3. 使用加密属性

```yaml
spring:
  datasource:
    password: '{cipher}4277f8d804f0e2286530d3855e8aceab5c8d75ac558bd731bbab45610f75f2a7'
```

> 在上面的这种配置下，REST 接口可以获得的便是解密后的信息，根本没有起到保密的效果！
>
> 此时，只能够保证敏感信息在 Git 仓库中的保密性，使得敏感信息能够在 Git 服务器能安全共享。
>
> <img src="../../../.gitbook/assets/image (13).png" alt="" data-size="original">\
>
>
> 增加配置 <mark style="color:orange;">**spring.cloud.config.server.encrypt.enabled=false**</mark>，可以解决上述问题，此时需要将解密的工作**交由 Config Client 完成**：
>
> ![](<../../../.gitbook/assets/image (14).png>)
