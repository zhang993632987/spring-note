# 添加国际化支持

## 1. 配置 locale

```properties
spring:
  web:
    locale: zh
    locale-resolver: accept_header
```

> spring boot 默认使用 <mark style="color:blue;">**AcceptHeaderLocaleResolver**</mark> 支持国际化。

## 2. 在 /src/main/resources 源文件夹下创建以下文件：

*   messages.properties

    ```properties
    license.create.message=%s
    license.update.message=%s
    license.delete.message=%s, %s
    ```
*   messages\_en.properties

    ```properties
    license.create.message=License created %s
    license.update.message=License %s updated
    license.delete.message=Deleting license with id %s for the organization %s
    ```
*   messages\_zh.properties

    ```properties
    license.create.message=许可证 %s 被创建
    license.update.message=许可证 %s 被更新
    license.delete.message=删除组织 %s 下的id为 %s 的许可证
    ```

## 3. 修改 Controller

```java
@PostMapping
public String createLicense(
        @PathVariable("organizationId") String organizationId,
        @RequestBody License license) {
    return licenseService.createLicense(license, organizationId);
}

@PutMapping
public String updateLicense(
        @PathVariable("organizationId") String organizationId,
        @RequestBody License license) {
    return licenseService.updateLicense(license, organizationId);
}

@DeleteMapping
public String deleteLicense(
        @PathVariable("organizationId") String organizationId,
        @PathVariable("licenseId") String licenseId) {
    return licenseService.deleteLicense(licenseId, organizationId);
}
```

## 4. 修改 Service 接口和实现

{% code overflow="wrap" %}
```java
@Autowired
private MessageSource messageSource;

public String createLicense(License license, String organizationId) {
    String responseMessage = null;
    if (license != null) {
        license.setOrganizationId(organizationId);
        responseMessage = String.format(
                messageSource.getMessage("license.create.message", null, LocaleContextHolder.getLocale()),
                license);
    }
    return responseMessage;
}

public String updateLicense(License license, String organizationId) {
    String responseMessage = null;
    if (license != null) {
        license.setOrganizationId(organizationId);
        responseMessage = String.format(
                messageSource.getMessage("license.update.message", null, LocaleContextHolder.getLocale()),
                license);
    }
    return responseMessage;
}

public String deleteLicense(String licenseId, String organizationId) {
    return String.format(
            messageSource.getMessage("license.delete.message", null, LocaleContextHolder.getLocale()),
            licenseId,
            organizationId);

}
```
{% endcode %}

## 5. 测试

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**国际化的关键在于 **<mark style="color:blue;">**Accept-Language**</mark>** 请求头！**
