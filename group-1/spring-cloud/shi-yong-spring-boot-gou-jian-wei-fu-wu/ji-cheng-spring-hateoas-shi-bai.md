# 集成Spring HATEOAS（失败）

<mark style="color:blue;">**HATEOAS**</mark> 是 Hypermedia as the Engine of Application State 的缩写，意思是<mark style="color:blue;">**超媒体即应用状态引擎**</mark>。

Spring HATEOAS 是一个小项目，它允许我们创建遵循 HATEOAS 原则的 API，显示给定资源的相关链接。<mark style="color:orange;">**HATEOAS 原则指出，API 应该通过返回每个服务响应可能的后续步骤的信息来为客户端提供指导。**</mark>这个项目不是核心功能或必备功能，但是如果你想拥有一个给定资源的所有 API 服务的完整指南，它是一个很好的选择。

## 1. 引入 Maven 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```

## 2. 实体类继承 RepresentationModel

```java
package com.study.cloudlearning.entity;

import lombok.Getter;
import lombok.Setter;
import lombok.ToString;
import org.springframework.hateoas.RepresentationModel;

/**
 * @author Zhang B H
 * @create 2023-10-26 10:13
 */
@Getter
@Setter
@ToString
public class License extends RepresentationModel<License> {
    private int id;
    private String licenseId;
    private String description;
    private String organizationId;
    private String productName;
    private String licenseType;
}

```

## 3. 修改Controller控制器

