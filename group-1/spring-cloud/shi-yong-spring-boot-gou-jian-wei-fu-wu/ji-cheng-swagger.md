# 集成swagger

参考knife4j的官网文档：[https://doc.xiaominfo.com/docs/quick-start](https://doc.xiaominfo.com/docs/quick-start)

## 1. 引入Maven依赖

```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
    <version>4.3.0</version>
</dependency>
```

## 2. 配置application.yml

```yaml
############################## Swagger ##############################
springdoc:
  swagger-ui:
    path: /swagger-ui.html
    tags-sorter: alpha
    operations-sorter: alpha
  api-docs:
    path: /v3/api-docs
  group-configs:
    - group: 'default'
      paths-to-match: '/**'
      packages-to-scan: com.study.license.controller
knife4j:
  enable: true
  setting:
    language: zh_cn
############################## Swagger ##############################
```

## 3. 测试

访问[http://localhost:8080/doc.html](http://localhost:8080/doc.html)
