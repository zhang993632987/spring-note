# 启用 Remember-me 功能

在 Spring Security 中为应用添加 Remember-me 功能只需**在 configure() 方法所传入的 HttpSecurity 对象上调用 rememberMe()** 即可。

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    .formLogin()
    .loginPage("/login");
    .and()
    .rememberMe()
    .tokenValiditySeconds(2419200)
    .key("spittrKey")
    ...
}
```

默认情况下，<mark style="color:blue;">**这个功能是通过在 cookie 中存储一个 token 完成的**</mark>，这个 token 最多两周内有效。在这里，我们指定这个 token 最多四周内有效（2,419,200秒）。

<mark style="color:blue;">**存储在 cookie 中的 token 包含用户名、密码、过期时间和一个私钥 —— 在写入 cookie 前都进行了 MD5 哈希**</mark>。默认情况下，<mark style="color:blue;">**私钥**</mark>的名为 SpringSecured，在这里我们将其设置为 spitterKey。

> Remember-me 功能已经启用，我们需要有一种方式来让用户表明他们希望应用程序能够记住他们。
>
> 为了实现这一点，<mark style="color:blue;">**登录请求必须包含一个名为 remember-me 的参数。在登录表单中，增加一个简单复选框就可以完成这件事情：**</mark>
>
> ```html
> <input id="remember_me" name="remember-me" type="checkbox" />
> <label for="remember_me" class="inline">Remember me</label>
> ```
