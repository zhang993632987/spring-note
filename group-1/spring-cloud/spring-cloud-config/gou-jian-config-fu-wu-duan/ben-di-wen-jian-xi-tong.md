# 本地文件系统

{% hint style="danger" %}
<mark style="color:red;">**不要在中大型云应用程序中使用基于文件系统的解决方案！**</mark>

使用文件系统方法，意味着要为想要访问应用程序配置数据的所有 Config 服务器端实现共享文件挂载点。
{% endhint %}

## 1. 修改 application.yml

在 application.yml 文件中增加相关配置信息，包括:

* **spring.profiles.active=native**
* **spring.cloud.config.server.native.search-locations=classpath:/config**

```yaml
server:
  port: 8071
spring:
  application:
    name: config-server
  profiles:
    active: native
  cloud:
    config:
      server:
        native:
          search-locations: classpath:/config
```

classpath 属性使得 Spring Cloud Config 服务器端在 **src/main/resources/config** 文件夹中查找配置数据。

## 2. 创建服务的配置文件

为以下 3 个环境创建应用程序配置数据：**默认环境**、**开发环境**和**生产环境**。

{% hint style="info" %}
应用程序配置文件的命名约定是“<mark style="color:orange;">**应用程序名称-环境名称.properties**</mark>”或“<mark style="color:orange;">**应用程序名称-环境名称.yml**</mark>”。环境名称由在服务启动时命令行传入的 Spring Boot 的 <mark style="color:blue;">**profile**</mark> 指定。
{% endhint %}

在 **src/main/resources/config** 文件夹分别创建了三个文件：

*   **license-service**.**yml**：

    ```yaml
    management:
      endpoints:
        web:
          base-path: /actuator
    server:
      port: 8080
    spring:
      web:
        locale: zh
        locale-resolver: accept_header
    ```
*   **license-service-dev.yml**：

    ```yaml
    management:
      endpoints:
        web:
          exposure:
            include: '*'
        jmx:
          exposure:
            include: '*'
    # springdoc-openapi项目配置
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
          packages-to-scan: com.study.cloudlearning.controller
    # knife4j的增强配置，不需要增强可以不配
    knife4j:
      enable: true
      setting:
        language: zh_cn
    ```
*   **license-service-prod.yml：**

    ```yaml
    management:
      endpoints:
        web:
          exposure:
            include: health
        jmx:
          exposure:
            exclude: '*'
    springdoc:
      api-docs:
        enabled: false
      swagger-ui:
        enabled: false
    knife4j:
      enable: false
    ```

## 3. 启动 Config Server

通过 REST 接口获取不同环境的配置信息：

<details>

<summary><mark style="color:purple;">http://localhost:8071/<strong>license-service</strong>/default</mark></summary>

```json
{
    "name": "cloud-learning",
    "profiles": [
        "default"
    ],
    "label": null,
    "version": null,
    "state": null,
    "propertySources": [
        {
            "name": "classpath:/config/cloud-learning.yml",
            "source": {
                "management.endpoints.web.base-path": "/actuator",
                "server.port": 8080,
                "spring.web.locale": "zh",
                "spring.web.locale-resolver": "accept_header"
            }
        }
    ]
}
```

</details>

<details>

<summary><mark style="color:purple;">http://localhost:8071/<strong>license-service</strong>/dev</mark></summary>

{% code overflow="wrap" %}
```json
{
    "name": "cloud-learning",
    "profiles": [
        "dev"
    ],
    "label": null,
    "version": null,
    "state": null,
    "propertySources": [
        {
            "name": "classpath:/config/cloud-learning-dev.yml",
            "source": {
                "management.endpoints.web.exposure.include": "*",
                "management.endpoints.jmx.exposure.include": "*",
                "knife4j.enable": true,
                "knife4j.openapi.description": "Swagger接口文档，学习用",
                "knife4j.openapi.title": "Swagger接口文档",
                "knife4j.openapi.group.test.api-rule": "package",
                "knife4j.openapi.group.test.api-rule-resources": "com.study.cloudlearning.controller",
                "knife4j.openapi.group.test.group-name": "测试分组"
            }
        },
        {
            "name": "classpath:/config/cloud-learning.yml",
            "source": {
                "management.endpoints.web.base-path": "/actuator",
                "server.port": 8080,
                "spring.web.locale": "zh",
                "spring.web.locale-resolver": "accept_header"
            }
        }
    ]
}
```
{% endcode %}

</details>

<details>

<summary><mark style="color:purple;">http://localhost:8071/<strong>license-service</strong>/prod</mark></summary>

```json
{
    "name": "cloud-learning",
    "profiles": [
        "prod"
    ],
    "label": null,
    "version": null,
    "state": null,
    "propertySources": [
        {
            "name": "classpath:/config/cloud-learning-prod.yml",
            "source": {
                "knife4j.enable": false,
                "management.endpoints.web.exposure.include": "health",
                "management.endpoints.jmx.exposure.exclude": "*"
            }
        },
        {
            "name": "classpath:/config/cloud-learning.yml",
            "source": {
                "management.endpoints.web.base-path": "/actuator",
                "server.port": 8080,
                "spring.web.locale": "zh",
                "spring.web.locale-resolver": "accept_header"
            }
        }
    ]
}
```

</details>

> 仔细观察，会看到在**选择 dev 端点**时，**Spring Cloud Config 服务器端返回的是**<mark style="color:blue;">**默认配置**</mark>**和**<mark style="color:blue;">**开发环境下的配置**</mark>。
>
> Spring Cloud Config 返回两组配置信息的原因是：Spring 框架实现了一种用于解决问题的层次结构机制。<mark style="color:orange;">**当 Spring 框架解决问题时，它将先查找默认属性文件中定义的属性，然后用特定环境的值（如果存在）去覆盖默认值。**</mark>
