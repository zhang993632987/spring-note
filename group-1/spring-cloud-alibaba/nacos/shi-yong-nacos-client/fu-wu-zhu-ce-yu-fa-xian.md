# 服务注册与发现

## 1. 添加依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    <version>${latest.version}</version>
</dependency>
```

## 2. 配置

在 application.yml 中配置 Nacos server 的地址：

```yaml
############################## 服务注册 ##############################
spring:
  cloud:
    nacos:
      discovery:
        username: nacos
        password: nacos
        server-addr: localhost:8848
        namespace: public
```

## 3. 通过 @EnableDiscoveryClient 开启服务注册发现功能

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosProviderApplication {

	public static void main(String[] args) {
		SpringApplication.run(NacosProviderApplication.class, args);
	}

	@RestController
	class EchoController {
		@RequestMapping(value = "/echo/{string}", method = RequestMethod.GET)
		public String echo(@PathVariable String string) {
			return "Hello Nacos Discovery " + string;
		}
	}
}
```

## 使用服务发现来查找服务

与使用 Eureka 时完全一样：

{% content-ref url="../../../spring-cloud/fu-wu-fa-xian-eureka/shi-yong-fu-wu-fa-xian-lai-cha-zhao-fu-wu/" %}
[shi-yong-fu-wu-fa-xian-lai-cha-zhao-fu-wu](../../../spring-cloud/fu-wu-fa-xian-eureka/shi-yong-fu-wu-fa-xian-lai-cha-zhao-fu-wu/)
{% endcontent-ref %}

