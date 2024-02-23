# 配置管理

## 1. 添加依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

## 2. 配置

在 **application.yml** 中配置 Nacos Server 的地址和应用名：

```yaml
spring:
  application:
    name: @artifactId@
  profiles:
    active: @profiles.active@

---
spring:
  config:
    import: nacos:${spring.application.name}.${spring.cloud.nacos.config.file-extension}?refresh=true
  cloud:
    nacos:
      config:
        username: nacos
        password: nacos
        server-addr: @nacos.addr@
        file-extension: yml
        namespace: @nacos.namespace@
```

<mark style="color:blue;">**spring.application.name 是构成 Nacos 配置管理 dataId 字段的一部分。**</mark>

{% hint style="success" %}
**在 Nacos Spring Cloud 中，dataId 的完整格式如下：**

```bash
${prefix}-${spring.profiles.active}.${file-extension}
```

* **prefix 默认为 spring.application.name 的值**，也可以通过配置项 spring.cloud.nacos.config.prefix来配置。
* spring.profiles.active 即为当前环境对应的 profile。**当 spring.profiles.active 为空时，对应的连接符 - 也将不存在，dataId 的拼接格式变成 ${prefix}.${file-extension}。**
* **file-exetension 为配置内容的数据格式**，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置。**目前只支持 properties 和 yaml 类型。**
{% endhint %}

{% hint style="warning" %}
## <mark style="color:orange;">注意</mark>

**与 Spring Cloud Config 一样，spring-cloud-starter-alibaba-nacos-config 在加载配置的时候，不仅仅加载了以 dataId 为 ${spring.application.name}.${file-extension:properties} 为前缀的基础配置，还加载了 dataId 为 ${spring.application.name}-${profile}.${file-extension:properties} 的基础配置。**
{% endhint %}

## 3. 通过注解 @RefreshScope 实现配置自动更新

```java
@RefreshScope
@RestController
@RequestMapping("/config")
public class ConfigController {

    @Value("${useLocalCache:false}")
    private boolean useLocalCache;

    @RequestMapping("/get")
    public boolean get() {
        return useLocalCache;
    }
}
```

## <mark style="color:orange;">注意</mark>

**如果想要完整地利用 nacos 相关的配置，必须将所有配置放在 **<mark style="color:orange;">**bootstrap.yml**</mark>** 或 **<mark style="color:orange;">**bootstrap.properties**</mark>** 文件中，并且必须引入 bootstrap 相关依赖（或者通过将系统环境变量 spring.cloud.bootstrap.enabled 设为 true）：**

```xml
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

<mark style="color:orange;">**只有这样，才能够省略 spring.config.import 配置，进而充分利用 nacos 特定的配置。**</mark>

> ## **spring cloud config 文档中的原文如下：**
>
> <mark style="background-color:blue;">Unless you are using</mark> <mark style="background-color:blue;"></mark><mark style="background-color:blue;">**config first bootstrap**</mark><mark style="background-color:blue;">, you will need to have a spring.config.import property in your configuration properties with an optional: prefix.</mark> For example:
>
> ```properties
> spring.config.import=optional:configserver:
> ```
>
> ### Config First Bootstrap <a href="#config-first-bootstrap" id="config-first-bootstrap"></a>
>
> To use the legacy bootstrap way of connecting to Config Server, <mark style="background-color:blue;">**bootstrap must be enabled via a property or the spring-cloud-starter-bootstrap starter.**</mark> The property is spring.cloud.bootstrap.enabled=true. It must be set as a System Property or environment variable.
