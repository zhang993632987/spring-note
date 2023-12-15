# 启用 HTTP Basic 认证

{% hint style="info" %}
<mark style="color:blue;">**HTTP Basic 认证（HTTP Basic Authentication）会直接通过 HTTP 请求本身，对要访问应用程序的用户进行认证。**</mark>

当在 Web 浏览器中使用时，它将向用户弹出一个简单的模态对话框。这只是 Web 浏览器的显示方式。**本质上，这是一个 HTTP 401 响应， 表明必须要在请求中包含一个用户名和密码。**
{% endhint %}

**如果要启用 HTTP Basic 认证的话，只需在 configure() 方法所传入的 HttpSecurity 对象上调用 httpBasic() 即可**。另外，还可以通过调用 realmName() 方法指定域。如下是在 Spring Security 中启用 HTTP Basic 认证的典型配置：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    .formLogin()
    .loginPage("/login");
    .and()
    .httpBasic()
    .realmName("Spittr")
    .and()
    ...
}
```

