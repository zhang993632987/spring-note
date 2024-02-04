# 配置管理

## 1. 添加依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

## 2. 配置

在 application.yml 中配置 Nacos Server 的地址和应用名：

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

**与 Spring Cloud Config 一样，spring-cloud-starter-alibaba-nacos-config 在加载配置的时候，不仅仅加载了以 dataId 为 ${spring.application.name}.${file-extension:properties} 为前缀的基础配置，还加载了dataId为 ${spring.application.name}-${profile}.${file-extension:properties} 的基础配置。**
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
