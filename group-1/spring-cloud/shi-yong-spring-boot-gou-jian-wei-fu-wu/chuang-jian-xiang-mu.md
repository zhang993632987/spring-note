# 创建项目

## 1. 新建项目

使用 Spring Initializer，选择 **Spring Boot DevTools**、**Lombok**、**Spring Web** 和 **Spring Boot Actuator**，生成项目的 pom.xml 的核心文件内容如下：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

## 2.创建实例类 License

```java
package com.study.license.entity;

import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

/**
 * @author Zhang B H
 * @create 2023-10-26 10:13
 */
@Getter
@Setter
@ToString
public class License {
    private int id;
    private String licenseId;
    private String description;
    private String organizationId;
    private String productName;
    private String licenseType;
    private Organization organization;
}
```

## 3. 创建服务接口及其实现类

```java
package com.study.license.service;

import com.study.license.entity.License;

/**
 * @author Zhang B H
 * @create 2023-10-26 10:14
 */
public interface LicenseService {

    License getLicense(String licenseId, String organizationId);

    String createLicense(License license, String organizationId);

    String updateLicense(License license, String organizationId);

    String deleteLicense(String licenseId, String organizationId);
}
```

{% code overflow="wrap" %}
```java
package com.study.license.service;

import com.study.cloudlearning.entity.License;
import org.springframework.stereotype.Service;

import java.util.Random;

/**
 * @author Zhang B H
 * @create 2023-10-26 10:16
 */
@Service
public class LicenseServiceImpl implements LicenseService {

    public License getLicense(String licenseId, String organizationId) {
        License license = new License();
        license.setId(new Random().nextInt(1000));
        license.setLicenseId(licenseId);
        license.setOrganizationId(organizationId);
        license.setDescription("Software product");
        license.setProductName("Ostock");
        license.setLicenseType("full");
        return license;
    }

    public String createLicense(License license, String organizationId) {
        String responseMessage = null;
        if (license != null) {
            license.setOrganizationId(organizationId);
            responseMessage = String.format("This is the post and the object is: %s ", license);
        }
        return responseMessage;
    }

    public String updateLicense(License license, String organizationId) {
        String responseMessage = null;
        if (license != null) {
            license.setOrganizationId(organizationId);
            responseMessage = String.format("This is the put and the object is: %s ", license);
        }
        return responseMessage;
    }

    public String deleteLicense(String licenseId, String organizationId) {
        return String.format(
                "Deleting license with id %s for the organization %s",
                licenseId,
                organizationId);
    }
}

```
{% endcode %}

## 4. 创建控制器 Controller

```java
package com.study.license.controller;

import com.study.cloudlearning.entity.License;
import com.study.cloudlearning.service.LicenseService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author Zhang B H
 * @create 2023-10-26 10:11
 */
@RestController
@RequestMapping("v1/organization/{organizationId}/license")
public class LicenseController {

    private LicenseService licenseService;

    @GetMapping("{licenseId}")
    public License getLicense(
            @PathVariable("organizationId")  String organizationId,
            @PathVariable("licenseId") String licenseId) {
        return licenseService.getLicense(licenseId, organizationId);
    }

    @Autowired
    public void setLicenseService(LicenseService licenseService) {
        this.licenseService = licenseService;
    }
}
```

## 5. 配置 application.yml

```properties
server:
  port: 8080

############################## Actuator ##############################
management:
  endpoints:
    web:
      exposure:
        include: '*'
    jmx:
      exposure:
        include: '*'
  endpoint:
    health:
      enabled: true
      show-details: always
############################## Actuator ##############################
```

## 6. 测试端点

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
