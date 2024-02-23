# 自定义 Group 的配置

**在没有指定 ${spring.cloud.nacos.config.group} 配置的情况下， 默认使用的是 DEFAULT\_GROUP。**如果需要自定义自己的 Group，可以通过以下配置来实现：

```properties
spring.cloud.nacos.config.group=DEVELOP_GROUP
```
