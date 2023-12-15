# 防止跨站请求伪造（CSRF）

{% hint style="info" %}
## <mark style="color:blue;">跨站请求伪造（cross-site request forgery，CSRF）</mark>

**如果一个站点欺骗用户提交请求到其他服务器的话，就会发生 CSRF 攻击。**
{% endhint %}

**从 Spring Security 3.2 开始，默认就会启用 CSRF 防护。**

<mark style="color:blue;">**Spring Security 通过一个同步 token 的方式来实现 CSRF 防护的功能：**</mark>

* **它将会拦截状态变化的请求（例如，非 GET、HEAD、OPTIONS 和 TRACE 的请求）并检查 CSRF token。**
* **如果请求中不包含 CSRF token 的 话，或者 token 不能与服务器端的 token 相匹配，请求将会失败，并抛出 CsrfException 异常。**

**这意味着在你的应用中，所有的表单必须在一个 “\_csrf” 域中提交 token，而且这个 token 必须要与服务器端计算并存储的 token 一致，这样的话当表单提交的时候，才能进行匹配。**

> 如果使用 **Thymeleaf** 作为页面模板的话，只要 \<form> 标签的 action 属性添加了 Thymeleaf 命名空间前缀，那么就会自动生成一 个 “\_csrf” 隐藏域：
>
> ```html
> <form method="POST" th:action="@{/spittles}">
>   ...
> </form>
> ```
>
> 如果使用 **JSP** 作为页面模板的话，要做的事情非常类似：
>
> ```html
> <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}" />
> ```

处理 CSRF 的另外一种方式就是根本不去处理它。我们可以在配置中**通过调用 csrf().disable() 禁用 Spring Security 的 CSRF 防护功能**， 如下所示：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    ...
    .csrf()
    .disable();
}
```
