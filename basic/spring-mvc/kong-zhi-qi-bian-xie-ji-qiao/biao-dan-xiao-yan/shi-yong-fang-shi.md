# 使用方式

## 1. 添加依赖

从 Spring 3.0 开 始，在 Spring MVC 中提供了对 Java 校验 API 的支持。**在 Spring MVC 中要使用 Java 校验 API 的话，并不需要什么额外的配置。只要保证在类路径下包含这个 Java API 的实现即可，比如 Hibernate Validator。**

```xml
<dependency>
  <groupId>org.hibernate.validator</groupId>
  <artifactId>hibernate-validator</artifactId>
  <version>6.2.5.Final</version>
</dependency>
```

## 2. 待校验的实体类&#x20;

将限制条件的注解添加到类上，如下所示：

```java
public class PageBounds {

    @Min(value=1, message = "页码必须大于等于1")
    private int page;

    @Min(value=5, message = "页面大小必须大于等于5")
    private int count;

}
```

## 3. 修改控制器中的方法

修改控制器中的方法，添加 <mark style="color:blue;">**@Valid**</mark> 注解和 <mark style="color:blue;">**Errors**</mark> 参数：

```java
@GetMapping("form")
public String form(
    @Valid PageBounds pageBounds,
    Errors errors) {
    if (errors.hasErrors())
        return "error";
    return pageBounds.getPage() + " : " + pageBounds.getCount();
}
```

如果有校验出现错误的话，那么这些错误可以通过 Errors 对象进行访问。<mark style="color:orange;">**Errors 参数要紧跟在带有 @Valid 注解的参数后面，@Valid 注解所标注的就是要检验的参数。**</mark>

```java
@Test
public void formError() throws Exception {
    mockMvc.perform(
        MockMvcRequestBuilders.get("/form")
            .queryParam("page", "1")
            .queryParam("count", "0")
    ).andExpect(MockMvcResultMatchers.content().string("error"));
}
```

{% hint style="info" %}
## <mark style="color:blue;">提示</mark>

**如果不添加 Errors 参数，当传入的参数存在校验错误时，将会抛出 BindException 或 MethodArgumentNotValidException，因此可以**<mark style="color:blue;">**使用 @ExceptionHandler 和 @ControllerAdvice 来完成统一的异常处理，返回合理的参数异常信息。**</mark>
{% endhint %}
